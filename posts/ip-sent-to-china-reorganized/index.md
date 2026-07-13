# IP 送中排查指南


「IP 送中」常被当成一个笼统故障，但真正排查时要拆开看：地理位置判断、线路路径、国内连通性和风控画像，分别对应不同的检测方法和处理策略。

&lt;!--more--&gt;

## 一 背景

「IP 送中」表面上是在说一个 IP 被识别成中国大陆，实际排障时不能这么粗糙地处理。它可能是 GeoIP 判断问题，也可能是线路绕路、国内连通性异常，或者 IP 本身风控身份太差。

更合理的拆法是把它分成四类：

1. GeoIP 被判成中国大陆；
2. 路由绕进中国大陆；
3. 国内访问已经被墙或半墙；
4. IP 风控身份太脏。

先分清是哪一类，再决定是检测、换 IP、改线路，还是长期养护。

## 二 先定义：什么叫 IP 送中

狭义的 IP 送中，通常指境外 VPS 或代理出口被某些服务识别成中国大陆 IP。这里最常见的是 Google 侧判断异常，比如搜索地区、YouTube Premium、Google Play、Gemini、Google AI Studio 等服务表现不对。

广义上，大家也会把「路由绕中」「IP 被墙」「风控脏 IP」一起叫送中。这样聊天方便，但排障时会误导。

比如：

- GeoIP 是 CN，问题在地理位置库或服务商自己的地区判断；
- 路由经过大陆运营商，问题在线路路径；
- 国内 TCP 端口大面积不通，问题可能是被墙；
- Scamalytics、IPQS、Spur 给高风险，问题是 IP 身份脏。

这四类问题的修法不一样。把它们混成一句「送中了」，后面就容易乱开药。

## 三 四类常见问题

### 1 GeoIP 送中

GeoIP 送中是最接近原意的一类：IP 地理位置库或目标服务自己的数据库，把你的境外 IP 判成了中国大陆。

常见表现：

- Google 搜索底部地区异常；
- YouTube Premium 提示所在国家或地区不支持；
- Google Play 地区错乱；
- Gemini 或 Google AI Studio 提示地区不可用；
- 一些订阅、支付、流媒体服务直接进入中国大陆规则。

先用几个接口做初筛：

```bash
curl -s https://ipinfo.io/json
curl -s https://api.ip.sb/geoip
curl -s https://www.cloudflare.com/cdn-cgi/trace | grep -E &#39;ip=|loc=|colo=&#39;
```

重点看：

```text
country
country_code
loc
```

Cloudflare trace 结果类似：

```text
ip=1.2.3.4
loc=US
colo=LAX
```

判断可以粗略这么看：

```text
loc=CN 或 country=CN
=&gt; 高度疑似 GeoIP 送中

多个数据库结果不一致
=&gt; 灰色状态，后续要测目标服务

Cloudflare 正常，但 Google 异常
=&gt; 不奇怪，说明两边用的不是同一套判断
```

这里有个坑：不要把 Cloudflare 的 `loc` 当成所有服务的最终答案。Cloudflare 认为是 US，不代表 Google 也认为是 US。实际要用什么服务，就要测什么服务。

### 2 路由送中

路由送中不是 IP 地理位置变了，而是网络路径不对。

比如你买的是日本、美国、新加坡 VPS，但去 Cloudflare、Google 或其他海外目标时，中间跳进了中国大陆运营商网络。这种情况下，GeoIP 可能还是国外，但延迟、丢包、可用性会明显变差。

用 `mtr` 看路径：

```bash
mtr -4ezbwrc 50 1.1.1.1
mtr -4ezbwrc 50 8.8.8.8
```

没有 `mtr` 时用：

```bash
traceroute -4n 1.1.1.1
traceroute -4n 8.8.8.8
```

重点看中间有没有这些运营商或 AS：

```text
China Telecom
China Unicom
China Mobile
AS4134
AS4837
AS4809
AS9808
AS58453
```

如果一台境外 VPS 访问海外目标，却绕进这些路径，就可以按路由异常处理。

这类问题通常不是靠「养 IP」解决，而是换线路、换机房、换上游，或者直接换服务商。

### 3 被墙或半墙

被墙和送中是两回事。

送中是国外服务把你当中国大陆 IP；被墙是中国大陆访问你这个 IP 不正常。两者可能同时发生，也可能完全无关。

检测国内可达性时，不要只看 ping。很多 VPS 本来就禁 ICMP，ping 不通不代表被墙。更应该测 TCP。

可以用这些工具：

- [ITDOG Ping](https://www.itdog.cn/ping/)
- [ITDOG TCPing](https://www.itdog.cn/tcping/)
- [ping.pe](https://ping.pe/)

建议测：

```text
IP
IP:22
IP:443
```

判断方式：

```text
国内 ping 大面积不通
=&gt; 只能作为参考，不能直接判死

国内 TCP 22/443 大面积不通，海外正常
=&gt; 更像被墙或半墙

只有部分运营商异常
=&gt; 可能是局部线路问题，也可能是封锁不完全
```

如果确认是被墙，IP-Sentinel 这类工具帮不上太多。它主要处理的是地理定位偏移和风控养护，不是 GFW 解封。

### 4 风控脏 IP

有些 IP 不送中，也没被墙，但还是难用。

典型表现是：

- 注册账号要求额外验证；
- 登录频繁触发安全检查；
- 支付失败；
- AI、流媒体、社交平台提示异常流量；
- 数据中心、代理、VPN 标签很重。

这时要查风控画像，而不是只查 GeoIP。

常用工具：

- [Scamalytics](https://scamalytics.com/)
- [IPQualityScore](https://www.ipqualityscore.com/free-ip-lookup-proxy-vpn-test)
- [Spur](https://spur.us/context/)
- [BrowserLeaks IP](https://browserleaks.com/ip)

重点看：

```text
proxy
vpn
hosting
risk score
recent abuse
datacenter abuse
```

如果结果里出现：

```text
High Risk
VPN/Proxy
Recently Abused
Datacenter Abuse
```

这个 IP 就算没被判成 CN，也不适合拿来做长期账号、支付、注册或关键服务入口。

这类问题最烦的是「邻居污染」。便宜 VPS、热门商家、共享出口，经常不是你一个人在用。你自己很干净，但同段其他人乱用，结果整个段被服务商打标签。

## 四 推荐检测顺序

实际排障时，不建议一上来就跑一堆脚本。按下面这个顺序更清楚。

### 1 先确认出口身份

```bash
curl -s https://www.cloudflare.com/cdn-cgi/trace | grep -E &#39;ip=|loc=|colo=&#39;
curl -s https://ipinfo.io/json
curl -s https://api.ip.sb/geoip
```

目标：判断是否明显 GeoIP 异常。

### 2 再确认线路路径

```bash
mtr -4ezbwrc 30 1.1.1.1
mtr -4ezbwrc 30 8.8.8.8
```

目标：判断是不是路由绕中国大陆。

### 3 然后确认国内连通性

用 ITDOG 或 ping.pe 测：

```text
IP:22
IP:443
```

目标：判断是不是被墙或半墙。

### 4 最后确认目标服务

如果你关心的是 Gemini，就测 Gemini。关心 YouTube Premium，就测 YouTube Premium。关心 Google Play，就测 Google Play。

通用 GeoIP 正常，不代表目标服务正常。这个结论看起来废话，但很多误判就出在这里。

## 五 IP-Sentinel 的定位

### 1 应该放在哪个位置

IP-Sentinel 是一个工具，但它不是普通检测脚本。

按照项目说明，IP-Sentinel 是一个分布式 VPS IP 养护系统，目标是处理 IP 定位偏移和风险分过高。它通过地理位置信号锚定、本地化访问行为、Telegram 管理面板和 Master-Agent 架构，长期维护一批 VPS 的 IP 状态。

它大概解决的是这个问题：

```text
我有多台 VPS，希望长期监控和维护它们在 Google 等服务眼里的地区身份。
```

它不是这个问题的最佳解：

```text
我临时买了一台小鸡，想知道它是不是送中。
```

临时检测，用前面的 GeoIP、路由、国内 TCP、风控分就够了。

### 2 技术定位

从项目结构看，IP-Sentinel 已经从单机脚本变成了 Master-Agent 架构：

```text
master/     控制中心，负责 Telegram 监听、Webhook 调度、SQLite 状态记录
core/       Agent 执行端，负责检测、锚定和任务执行
scripts/    指纹、UA 等生成逻辑
data/       地区规则、关键词、LBS 锚点、UA 数据
telemetry/  Cloudflare Workers 计数网关
```

它更像「节点资产养护系统」，不是「一键检测工具」。

适合：

- 有多台 VPS；
- 长期使用 Google、YouTube、Gemini 等地区敏感服务；
- 希望统一监控 IP 趋势；
- 能接受 Telegram Bot、Agent、后台任务的维护成本。

不适合：

- 只测一台 VPS；
- IP 已经被墙；
- 想马上把送中的 IP 拉回来；
- 不想维护额外服务。

我的态度还是保守：可以研究，但别当神药。

Google 的地区和风控判断不只看访问行为，还可能结合账号历史、设备定位、浏览器权限、IP 段邻居行为和数据库更新。模拟本地化流量也许有帮助，但不能保证稳定纠偏。

## 六 处理策略

分清类型后，处理就简单多了。

### 1 GeoIP 送中

优先级：

1. 换干净 IP；
2. 换更少人滥用的服务商；
3. 如果是自有 IP 段，再考虑 GeoFeed、数据库修正；
4. 长期多节点场景，再考虑 IP-Sentinel。

### 2 路由送中

优先级：

1. 换线路；
2. 换机房；
3. 换服务商；
4. 避免把路由问题当成 GeoIP 问题处理。

### 3 被墙或半墙

优先级：

1. 先确认 TCP 可达性；
2. 尝试换端口或入口；
3. 严重时换 IP；
4. 不要指望 IP 养护工具解封。

### 4 风控脏 IP

优先级：

1. 不拿它做重要账号；
2. 避免注册、支付、长期登录；
3. 换低滥用 IP 段；
4. 多节点长期使用时再考虑养护。

## 七 小结

「IP 送中」不是一个足够精确的故障描述。真正排查时，应该先问：

```text
是 GeoIP 被判 CN？
是路由绕中国大陆？
是国内访问被墙？
还是 IP 风控身份太脏？
```

我的建议是：单台 VPS 先按四层检测排除问题，坏了就换，别恋战。多节点长期使用，才值得考虑 IP-Sentinel 这类养护系统。

便宜小鸡可以随便换，账号和时间不该陪它耗着。

## 八 参考

- [IP-Sentinel GitHub 项目](https://github.com/hotyue/IP-Sentinel)
- [Cloudflare trace](https://www.cloudflare.com/cdn-cgi/trace)
- [ipinfo.io](https://ipinfo.io/)
- [ip.sb GeoIP](https://api.ip.sb/geoip)
- [ITDOG Ping](https://www.itdog.cn/ping/)
- [ITDOG TCPing](https://www.itdog.cn/tcping/)
- [ping.pe](https://ping.pe/)
- [Scamalytics](https://scamalytics.com/)
- [IPQualityScore](https://www.ipqualityscore.com/free-ip-lookup-proxy-vpn-test)
- [Spur](https://spur.us/context/)
- [BrowserLeaks IP](https://browserleaks.com/ip)


---

> 作者: [枫](https://github.com/qiuzhi)  
> URL: https://blog.iqzhi.com/posts/ip-sent-to-china-reorganized/  

