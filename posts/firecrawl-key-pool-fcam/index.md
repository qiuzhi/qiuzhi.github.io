# Firecrawl 多账号聚合折腾记


Hermes 的 `web_extract` 可以用 Firecrawl 抽取网页正文，但一个 Key 的额度和并发终究有限。我手里有几个自己使用的 Firecrawl 账号，与其把 Key 分散塞进不同应用，不如在前面加一层统一入口。

这次最后没有自建完整的 Firecrawl，也没有直接使用最轻量的 Key Rotator，而是部署了 FCAM：单容器、SQLite、内存状态，集中管理 Key，再给 Hermes 发一个 Client Token。中间还顺手找出了一个额度汇总的计算 Bug。

&lt;!--more--&gt;

## 一 先弄清楚我要自建什么

刚开始搜索“Firecrawl 自建”时，很容易把三件事混在一起：

1. **自建 Firecrawl 服务**：在自己的服务器上运行抓取 API、Worker、Playwright、Redis 等组件，不再消耗 Firecrawl Cloud Credits；
2. **Firecrawl Cloud Key 网关**：自己不抓网页，只管理多个 Cloud API Key，再把请求转发到 `api.firecrawl.dev`；
3. **Firecrawl 兼容服务**：第三方提供与 Firecrawl 相似的接口，但后端不是官方 Cloud，也不是官方账号池。

我的目标属于第二种。我只是想给 Hermes 的网页提取准备一个稳定入口，没有打算自己维护浏览器池和反爬代理。

官方 Firecrawl 确实支持自建，但当前完整栈并不算轻。除了 API 和 Worker，还涉及 Playwright、Redis、RabbitMQ、PostgreSQL 等组件。更重要的是，自建版没有 Cloud 的 Fire-engine 和托管反封锁能力。普通网页没有问题，碰到复杂风控站点，还得继续准备代理和浏览器环境。

为了一个个人用的网页提取入口，把整套抓取基础设施搬回来，投入和收益不太匹配，所以很快排除了完整自建。

## 二 在 Rotator 和 FCAM 之间做选择

找到的两个项目比较有代表性：

### 1 firecrawl-rotator

`xyonium/firecrawl-rotator` 是一个很小的 Go 反向代理，只依赖标准库，最终镜像也很轻。它会读取每把 Key 的剩余额度，优先选择额度更多的 Key，并对 `402`、`429`、`401` 等情况做切换。

从纯转发能力看，它其实更符合“Key 池”的直觉：

- 按剩余 Credits 选 Key；
- 额度不足时停用并等待对应账号的账期重置；
- 网络错误和部分 5xx 先重试；
- 部署简单，资源占用低。

但它更像一段专用基础设施，而不是完整的管理系统。入口鉴权、管理界面、审计日志和 Client 隔离都比较弱，`/status` 之类的状态接口也不能随便暴露公网。项目在我检查时刚出现一周左右，提交很集中，但还不足以证明长期维护稳定。

### 2 Firecrawl API Manager

FCAM 的仓库名是 `firecrawl-manger`，少了一个 `a`，但项目本身做得比名字完整。它提供：

- WebUI 管理 Key 和 Client；
- Client Token 鉴权；
- Key 加密存储；
- 每日配额、RPM 和并发控制；
- 请求日志与审计；
- Firecrawl v1、v2 兼容入口；
- 异步任务与创建任务所用 Key 的绑定。

它的缺点也很明确。当前 Key 选择只是普通轮询，不会优先选择剩余额度更多的账号；转发代码没有单独处理 `402 Payment Required`；每个 Key 也不能配置独立出站代理。README 里的“智能负载均衡”说得比较满，源码里的实现要朴素得多。

最后我还是选了 FCAM。原因不是它的转发策略更聪明，而是我的实际请求只有 Hermes 的 `POST /v2/scrape`，异步任务亲和性暂时用不上，反而是 Client Token、WebUI 和请求日志更有价值。

## 三 单容器部署 FCAM

FCAM 官方 Compose 给了 PostgreSQL 和 Redis 的生产模板，第一眼看起来又是一套不小的服务。继续检查配置后发现，个人单实例根本不需要这些组件：

- SQLite 保存 Client、Key、日志、配额和额度快照；
- `state.mode=memory` 保存限流、并发和冷却状态；
- PostgreSQL 和 Redis 主要服务于多实例与高可用。

我的部署机是 AMD64，所以直接使用发布的镜像，并固定到检查过的提交标签，不跟随 `latest`：

```yaml
services:
  fcam:
    image: guangshanshui/firecrawl-manager:sha-89d5d65
    container_name: firecrawl-manager
    restart: unless-stopped
    ports:
      - &#34;${FCAM_BIND_ADDR:-127.0.0.1}:8000:8000&#34;
    environment:
      FCAM_ADMIN_TOKEN: &#34;${FCAM_ADMIN_TOKEN}&#34;
      FCAM_MASTER_KEY: &#34;${FCAM_MASTER_KEY}&#34;
      FCAM_DATABASE__PATH: &#34;/app/data/api_manager.db&#34;
      FCAM_STATE__MODE: &#34;memory&#34;
      FCAM_SERVER__ENABLE_DOCS: &#34;false&#34;
    volumes:
      - &#34;./data:/app/data&#34;
```

配套的 `.env`：

```env
FCAM_BIND_ADDR=127.0.0.1
FCAM_ADMIN_TOKEN=替换为随机管理员Token
FCAM_MASTER_KEY=替换为至少32字节且长期固定的密钥
```

两个随机值可以这样生成：

```bash
openssl rand -hex 32
```

`FCAM_MASTER_KEY` 用于加密数据库里的 Firecrawl Key，部署后必须保持稳定。随便换掉它，旧 Key 就无法解密。

如果 Hermes 和 FCAM 不在同一台机器，可以把端口绑定到 EasyTier 等私网地址，再让 Hermes 通过私网访问。管理界面、Admin Token 和状态接口都没有直接暴露到公网。

SQLite 数据目录还要注意权限。容器使用非 root 用户运行，宿主机目录没有写权限时会出现 `sqlite3.OperationalError: unable to open database file`。可以创建目录后调整属主：

```bash
mkdir -p data
chown -R 10001:10001 data
```

启动并检查：

```bash
docker compose up -d
docker compose logs -f firecrawl-manager
curl http://127.0.0.1:8000/healthz
curl http://127.0.0.1:8000/readyz
```

两个健康接口都返回 `200` 后，再打开：

```text
http://FCAM地址:8000/ui2/
```

## 四 配置 Client 和 Firecrawl Key

FCAM 里的 Client 不是一个 Firecrawl 账号，而是调用 FCAM 的下游应用。Key 会按 Client 分组，一个 Client Token 只能使用分配给它的 Key。

我只给 Hermes 创建一个 Client：

```text
Name：hermes
Daily Quota：留空
Rate Limit / Min：30
Max Concurrent：6
Active：开启
```

这里按三个相互独立的 Free Team 估算。Firecrawl 当前 Free 计划的 `/scrape` 限制是每分钟 10 次、并发 2，因此外层 Client 上限取 `3 × 10 RPM` 和 `3 × 2 并发`。

Client 的 `Daily Quota` 留空代表不额外设置日上限，填 `0` 则是每天零请求，不是无限制。

每个 Firecrawl Key 都绑定到这个 `hermes` Client：

```text
Client：hermes
Provider：firecrawl
Plan Type：free
Max Concurrent：2
Rate Limit / Min：10
Active：开启
```

Key 的 `Daily Quota` 是 FCAM 统计的成功请求次数，不等于 Firecrawl Credits。我给它设置了一个宽松上限，主要是避免保留默认的每天 5 次。真正的套餐额度仍由 Firecrawl 上游控制。

还有一个容易误解的点：Firecrawl 的限流和 Credits 按 Team 计算。同一个 Team 里创建多把 API Key，并不会让额度和吞吐量翻倍。

全部 Key 分配完成后，FCAM 生成一个 Client Token。Hermes 只需要保存这个 Token，不需要知道后面的 Firecrawl Key。

## 五 接入 Hermes

Hermes 支持把搜索和网页提取拆给不同提供商。我保留 Tavily 负责搜索，只把提取交给 Firecrawl：

```yaml
web:
  search_backend: tavily
  extract_backend: firecrawl
```

在 Hermes 的 `.env` 里加入两项：

```env
FIRECRAWL_API_URL=http://FCAM私网地址:8000
FIRECRAWL_API_KEY=FCAM生成的Client-Token
```

`FIRECRAWL_API_URL` 只写根地址，不要手工追加 `/v2`。Hermes 使用 Firecrawl SDK，调用 `web_extract` 时会自动请求：

```text
POST /v2/scrape
```

这里的 `FIRECRAWL_API_KEY` 也不是 Firecrawl 原始 Key，而是 FCAM 的 Client Token。保存后重启 Hermes Gateway，链路就变成：

```text
Hermes web_extract
        ↓
FCAM Client Token 鉴权
        ↓
FCAM 轮询选择 Firecrawl Key
        ↓
api.firecrawl.dev/v2/scrape
```

## 六 验证时多花了一个 Credit

第一次从 Hermes 请求 FCAM 时遇到了：

```text
No route to host
```

FCAM 的健康状态正常，TCP 端口稍后又能连上，最后确认是私网路由在服务重启后短暂没有收敛，并不是 Client Token 或 Firecrawl Key 配错。

为了分层排查，当时做了两次成功的 Scrape：

1. 直接请求 FCAM 的 `/v2/scrape`，确认 Client Token、Key 池和上游正常；
2. 再调用 Hermes 的 `web_extract`，确认实际工具链路正常。

两次都抓取 `https://example.com`，最终返回标题 `Example Domain` 和对应 Markdown。这也意味着验证实际消耗了两个 Credits。

单纯确认接入是否成功，一次真实的 `web_extract` 已经足够。直接探测加工具调用适合定位“FCAM 不通”还是“Hermes 配置不对”，没必要每次都跑两层。

## 七 额度汇总出现负数

服务跑通后，FCAM 的 Client 面板出现了一个很奇怪的数字：

```text
剩余 3,070 / 3,000
已用 -2.33%
```

刚开始以为是 Key 参数填错，顺着代码查下去，发现是项目自己的统计口径有问题。

Firecrawl 的额度接口会分别返回：

```json
{
  &#34;remainingCredits&#34;: 1024,
  &#34;planCredits&#34;: 1000
}
```

`planCredits` 是套餐基础额度，`remainingCredits` 还可能包含赠送或额外 Credits。三个账号的初始可用余额正好可能是：

```text
3 × 1024 = 3072
```

前面两次验证各消耗一个 Credit，于是剩下：

```text
3072 - 2 = 3070
```

这个数字与面板完全吻合。真实余额没有错，错的是 FCAM 把三个账号的套餐额度 `3000` 当成完整总额度，然后计算：

```python
usage = (plan_credits - remaining_credits) / plan_credits * 100
```

代入数值就是：

```text
(3000 - 3070) ÷ 3000 × 100 = -2.33%
```

代码里，单个 Key 的显示会读取第一条成功额度快照，并把百分比限制在 `0～100%`；Client 汇总却优先使用 `planCredits`，也没有处理负数。同一个项目里，两套显示逻辑并不一致，现有测试又只覆盖了“剩余额度小于套餐额度”的正常场景。

这属于统计 Bug，主要影响面板显示和额度刷新频率，不影响 Client 鉴权、Scrape 请求和 Key 轮询。更合理的修法是把套餐额度和额外额度分开显示；至少也应该从套餐额度、第一条成功快照和当前余额中取最大值作为有效总额，并对百分比做边界限制。

按当前数据，合理的显示应该接近：

```text
剩余 3,070 / 初始可用 3,072
已用 0.07%
```

## 八 当前方案的边界

FCAM 已经满足我现在的使用需求，但它不是把多个 Key 塞进去就万事大吉。

### 1 它不是按额度调度

当前 Key Pool 是普通轮询。额度监控用于展示和定时刷新，不会优先选择剩余 Credits 最多的 Key。想要真正的额度感知调度，`firecrawl-rotator` 反而更直接。

### 2 没有每个 Key 独立代理

FCAM 的数据库、管理 API 和 WebUI 都没有 `proxy_url` 字段。可以通过 `HTTP_PROXY`、`HTTPS_PROXY` 给整个容器设置全局代理，但所有 Key 会共用同一个出口。

而且这个代理只改变：

```text
FCAM → Firecrawl Cloud
```

目标网站看到的仍然是 Firecrawl Cloud 的抓取出口，不是 FCAM 所在服务器或它配置的代理。

### 3 额度耗尽需要继续观察

当前转发代码会处理 `429`、`401`、`403` 和部分 5xx，但没有明确的 `402` 自动切换分支。免费账号真正耗尽时，是否能按预期切到下一把 Key，还需要用实际响应验证。

### 4 项目还不算成熟基础设施

FCAM 的管理能力不错，但最后一次提交距离我部署时已有几个月。镜像应固定版本，不适合无脑跟随 `latest`。WebUI 的额度负数也说明 README 里的功能描述不能替代源码检查和真实验证。

另外，聚合自己持有或明确授权的 Key，与批量注册免费账号绕过平台额度是两回事。后者容易触发风控，也不是一个适合长期维护的方案。

## 九 最后的落地方案

这次没有追求最完整的架构，最后留下的是一套很小的链路：

```text
Tavily：负责 web_search
Firecrawl：负责网页抓取
FCAM：管理 Firecrawl Key 和 Client Token
Hermes：统一调用 web_extract
EasyTier：提供 Hermes 到 FCAM 的私网连接
SQLite：保存 FCAM 配置、日志和额度快照
```

FCAM 单容器运行，不加 PostgreSQL 和 Redis；Hermes 只保存 FCAM Client Token；管理端和数据入口走私网；搜索继续交给 Tavily，避免把 Firecrawl Credits 花在不必要的搜索上。

这套方案不完美：轮询不看余额、没有 per-key 代理、额度统计还有 Bug。但对我现在的 `POST /v2/scrape` 场景已经够用。比起再搭一套完整 Firecrawl，维护一个单容器网关更符合实际需求。

## 十 参考

- [Firecrawl API Manager](https://github.com/ZeroPointSix/firecrawl-manger)
- [firecrawl-rotator](https://github.com/xyonium/firecrawl-rotator)
- [Firecrawl 文档：Rate Limits](https://docs.firecrawl.dev/rate-limits)
- [Firecrawl 文档：Self-hosting](https://docs.firecrawl.dev/contributing/self-host)
- [Hermes Agent 文档：Web Search &amp; Extract](https://hermes-agent.nousresearch.com/docs/user-guide/features/web-search)
- [FCAM Docker Hub](https://hub.docker.com/r/guangshanshui/firecrawl-manager)


---

> 作者: [枫](https://github.com/qiuzhi)  
> URL: https://blog.iqzhi.com/posts/firecrawl-key-pool-fcam/  

