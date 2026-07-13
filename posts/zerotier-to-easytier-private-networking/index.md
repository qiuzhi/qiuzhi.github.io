# 从 ZeroTier 到 EasyTier 的异地组网实践


我原本已经在用 ZeroTier，而且整体稳定。这次折腾并不是因为它突然不能用了，而是想给家里的 NAS、服务器和移动设备多准备一条可控的异地访问路径，顺便弄清楚 Tailscale、ZeroTier、EasyTier 到底差在哪里。

最后没有做激进迁移：ZeroTier 继续保留，EasyTier 用国内 VPS 搭了一套私有中继，作为备用链路和后续扩展的底座。

&lt;!--more--&gt;

## 一 为什么又折腾一套组网

我的需求不复杂：人在外面时访问家里的 NAS、Kavita、SSH 和其他 Web 服务，最好还能让 OpenWrt 作为家庭局域网入口，把没有安装客户端的设备一并带进来。

这类需求本质上是三层网络访问，不依赖局域网广播、ARP 或同网段发现，所以没有必须保留二层虚拟以太网的刚需。真正需要解决的是三件事：

1. 两端能否成功 P2P 直连；
2. 打洞失败后有没有稳定的 Relay；
3. 中继路径、密钥和带宽是否掌握在自己手里。

我最初的问题是“哪一个更好”，后来发现这个问法太宽。只要能直连，三者的数据都不会长期经过厂商或自建中继，实际速度主要取决于两端网络。差异真正出现的地方，是打洞失败之后。

## 二 Tailscale、ZeroTier 与 EasyTier 怎么选

### 1 Tailscale

Tailscale 的优势是省心。它基于 WireGuard，客户端成熟，账号登录、设备管理、ACL 和 MagicDNS 都做得很完整。两台设备可以直连时，体验往往是三者里最顺滑的。

它的问题也很明确：如果直连失败，流量会转到 DERP。DERP 解决的是可达性，不保证路线一定最短。自建 DERP 可以增强控制力，但意味着证书、域名、升级和监控都要自己维护。

如果从零开始，只想快速连几台设备，我仍然会优先推荐 Tailscale。

### 2 ZeroTier

ZeroTier 是我原本就在用的方案。它提供类似虚拟交换机的网络，节点通过 UDP 联系根节点、控制器和其他 Peer，完成发现、授权、打洞和数据传输。正常情况下业务流量也是端到端直连。

ZeroTier 的 Moon 经常被误解成“自建中继”。实际上 Moon 更接近自定义根节点，主要帮助节点发现和路径建立，不等同于一个可控的通用数据面 Relay。打洞失败时，是否能获得理想的中继路径，不是部署一个 Moon 就能完全解决的。

不过，已经稳定运行的网络没有必要为了追新而迁移。能稳定访问，比架构图看起来更漂亮重要。

### 3 EasyTier

EasyTier 最吸引我的地方，是它能比较直接地自建共享节点，并在 P2P 失败时承担真实业务流量中继。TCP、UDP、WS、WSS 等传输也给受限网络留下了更多选择。

代价是自己负责部署、安全和升级。它不像 Tailscale 那样开箱即用，但路径更可控，特别适合手里已经有国内 VPS、愿意自己维护 Docker 服务的人。

我的最终选择是：ZeroTier 不动，EasyTier 双跑。先验证一段时间，再决定它是长期备用还是逐步接管更多设备。

## 三 搭建国内 EasyTier 私有中继

### 1 网络规划

我给 EasyTier 规划了一个不容易和家庭 LAN、公司网络、Docker 网段冲突的地址段：

```text
EasyTier 虚拟网段：10.203.77.0/24
VPS 虚拟地址：    10.203.77.1/24
OpenWrt：          10.203.77.10/24
```

VPS 有公网 IPv4 即可，IPv6 不是前置条件。没有 IPv6 不需要特意加入 `disable_ipv6`；保留默认行为，未来增加双栈路径时更省事。

监听端口没有使用默认值，而是改成了自定义高位端口 `28473`，同时开放 TCP 和 UDP：

```text
28473/TCP
28473/UDP
```

改端口只能减少默认端口扫描和误碰撞，不是协议伪装，也不能保证绕过 DPI。

### 2 为什么必须使用私有模式

EasyTier 官方文档说明，不带参数启动的节点可以作为公共共享节点，默认节点也可能为其他虚拟网络提供转发。自己的 VPS 显然不该变成陌生人的免费中继，所以至少要配置：

```text
非显眼的网络名
足够长的随机密钥
private_mode = true
```

RPC 管理接口只绑定本机：

```text
127.0.0.1:15888
```

它只服务于本机的 `easytier-cli` 和管理操作，与 P2P、Relay 及业务流量无关，不应该暴露到公网。

`relay_network_whitelist = &#34;&#34;` 与 `relay_all_peer_rpc = true` 主要用于限制其他虚拟网络的数据转发、只保留发现和建链协助。私有模式已经把节点收敛到同名、同密钥网络，我的场景不需要再堆这组参数。

### 3 Docker 与配置文件

最初可以直接把参数写进 Compose 的 `command`，但参数一多，可读性很快下降。后来改成挂载 TOML 配置，让 Compose 只负责容器生命周期：

```yaml
services:
  easytier:
    image: easytier/easytier:v2.6.4
    container_name: easytier
    hostname: gz-vps
    restart: unless-stopped
    network_mode: host

    cap_add:
      - NET_ADMIN
      - NET_RAW

    devices:
      - /dev/net/tun:/dev/net/tun

    volumes:
      - /etc/machine-id:/etc/machine-id:ro
      - ./conf/easytier.toml:/config/easytier.toml:ro

    command:
      - -c
      - /config/easytier.toml
```

配置文件只读挂载，密钥使用占位符表示：

```toml
instance_name = &#34;home&#34;
hostname = &#34;gz-vps&#34;
ipv4 = &#34;10.203.77.1/24&#34;
dhcp = false

listeners = [
  &#34;tcp://0.0.0.0:28473&#34;,
  &#34;udp://0.0.0.0:28473&#34;,
]

rpc_portal = &#34;127.0.0.1:15888&#34;

[network_identity]
network_name = &#34;feng-private-net&#34;
network_secret = &#34;替换为足够长的随机密钥&#34;

[flags]
private_mode = true
```

EasyTier 官方支持通过多个 `-c` 加载多份配置，在同一进程中启动多个虚拟网络。如果更喜欢纯命令行，也可以运行多个容器或多个进程，每个私网使用独立的网络名、密钥、监听端口、RPC 端口和虚拟网段。

`instance_name` 只是用来区分同一台机器上的多个 EasyTier 实例，不需要和其他节点一致，也不强制与配置文件名一致。不过把 `home.toml` 对应到 `instance_name = &#34;home&#34;`，以后维护会轻松很多。

## 四 OpenWrt 和安卓接入

### 1 OpenWrt 作为家庭入口

OpenWrt 可以通过 `luci-app-easytier` 加入虚拟网络。最小配置只需要填写：

```text
虚拟 IP：10.203.77.10/24
网络名：feng-private-net
网络密钥：与 VPS 完全一致
Peer：tcp://VPS公网IP:28473
Peer：udp://VPS公网IP:28473
私有模式：开启
```

先只验证 OpenWrt 自己能否访问：

```bash
ping 10.203.77.1
```

确认链路正常后，再发布家庭 LAN 网段，例如 `192.168.50.0/24`。这样远端设备才能通过 OpenWrt 访问没有安装 EasyTier 的 NAS、电视或其他主机。基本链路和子网代理分两步做，比一次改完所有路由、防火墙更容易排错。

### 2 安卓客户端

Android 客户端可以从 EasyTier 官方 GitHub Releases 下载。大多数近年的手机选择 ARM64 APK。安装后需要允许应用创建 VPN 连接，这是建立系统虚拟网卡所需的正常权限。

移动网络的 NAT 和运营商策略变化更大，安卓端既适合验证 UDP P2P，也适合观察打洞失败后是否切换到 Relay。

## 五 TCP、UDP、WS 与 WSS

### 1 TCP 与 UDP

UDP 是打洞和低延迟传输的主力。它没有 TCP 的队头阻塞，网络状况正常时更适合远程桌面、视频和持续传输。

TCP 主要负责 UDP 受限时兜底。缺点是虚拟网络里的 TCP 再套一层 TCP，外层丢包重传时可能放大卡顿。

我最后保留 TCP 与 UDP 同时监听，而不是押注单一协议。

### 2 WS 与 WSS

WS 是 WebSocket over TCP，WSS 则在外面再套一层 TLS：

```text
WS： EasyTier → WebSocket → TCP
WSS：EasyTier → WebSocket → TLS → TCP
```

EasyTier 自身默认已有加密，所以 WS 并不等于业务内容明文传输；WSS 的价值更多是兼容 HTTPS、证书和反向代理。在只允许 HTTPS/WebSocket 的公司网或校园网里，WSS 可能比原生 TCP、UDP 更容易建立连接。

但 WSS 不会提升 P2P 打洞能力，也不保证无法被识别。它更适合作为严格网络下的备用入口，而不是替代 UDP。

### 3 多协议不是固定排队

同时配置多个 Listener，只代表服务器同时开放多个入口。客户端配置多个 Peer 后，也不能简单理解成“先 UDP，失败后 TCP，最后 WSS”。EasyTier 会建立和维护可用路径，并根据拓扑、路由 cost 和延迟选择实际链路。

典型过程可能是先通过 Relay 通信，路由 cost 为 `2`；P2P 建立后切换到直连，cost 变为 `1`。服务器开放 WSS，不代表客户端会自动猜到这个入口，客户端仍需配置对应的 `wss://` Peer。

## 六 打洞失败后 Relay 走哪里

EasyTier 没有另开一组类似 TURN 的中继端口。P2P 失败后，业务流量直接复用节点与 VPS 已经建立的连接。

例如两端可以分别通过不同传输接入：

```text
安卓 ──UDP:28473──&gt; VPS Relay ──TCP:28473──&gt; OpenWrt
```

VPS 只需开放配置中的监听端口；RPC 的 `15888` 不参与中继。P2P 打洞阶段可能出现由 NAT 动态分配的公网端口，但这不意味着 VPS 防火墙需要开放一大片随机入站端口。

是否真的走中继，不能靠感觉判断。应查看：

```bash
docker exec easytier easytier-cli peer
docker exec easytier easytier-cli route
```

直连成功时，VPS 基本不承载两端业务流量；只有 Relay 生效时，VPS 带宽和线路才成为瓶颈。

## 七 多个私网怎么处理

一台 VPS 可以服务多个独立私网，但不要用一套网络名和密钥混在一起。每个私网应有独立的：

- `instance_name`；
- `network_name` 与 `network_secret`；
- TCP/UDP 监听端口；
- RPC 端口；
- 虚拟网段。

配置文件方式可以重复传入 `-c`：

```bash
easytier-core \
  -c /config/home.toml \
  -c /config/work.toml
```

命令行方式则更适合一网一容器。两种方法都能工作，区别只是运维方式。配置文件更适合长期维护；命令参数更直观，但密钥会出现在 Compose 和 `docker inspect` 中。

还要注意，VPS 同时加入多个私网后，监听在 `0.0.0.0` 的业务服务可能被多个网络访问。真正需要隔离时，服务应绑定到指定虚拟 IP，或在主机防火墙中按接口和源网段限制。

## 八 国内和跨境路径的判断

国内设备连接国内 VPS，通常不经过国际出口意义上的 GFW。使用国内 EasyTier 中继的价值不是“协议隐身”，而是中继位置和带宽可控。

连接国外 VPS 时，无论 Tailscale、ZeroTier 还是 EasyTier，跨境链路都不会凭空消失。WSS 相比原生 UDP/TCP 更像普通 TLS 长连接，但目标 IP、连接时长、流量大小和行为特征仍然可见。换端口、套 WSS 都不能当作抗识别保证。

如果从零搭建少量国外节点，我会选 Tailscale，省心优先；但在已经部署 EasyTier 国内中继的前提下，直接让国外 VPS 加入现有 EasyTier 私网更合理，同时保留 ZeroTier 作为备用。真正决定体验的，通常还是 VPS 地区、运营商路由、丢包和晚高峰拥塞。

## 九 最后的选择

这次折腾最后没有得出“某一个软件全面碾压另外两个”的结论：

- Tailscale 赢在产品完成度和省心；
- ZeroTier 已经稳定时，没有迁移的必要；
- EasyTier 适合愿意自建中继、想控制路径的人。

我的落地方案是 ZeroTier 保留，EasyTier 以国内 VPS 私有中继的形式并行运行。TCP 与 UDP 作为主入口，WSS 只在确实遇到严格网络时再加；RPC 只监听本机，网络使用强密钥和私有模式；先验证 P2P 与 Relay，再逐步接入 OpenWrt、安卓和家庭子网。

折腾组网最容易犯的错，是还没测清实际路径，就开始堆端口、协议和参数。更稳的顺序始终是：先建立最小可用链路，再看 `peer` 和 `route`，最后只针对真实失败点调整。

## 十 参考

- [Tailscale 文档：Connection types](https://tailscale.com/kb/1257/connection-types)
- [Tailscale 文档：Custom DERP servers](https://tailscale.com/kb/1118/custom-derp-servers)
- [ZeroTier 文档：The Protocol](https://docs.zerotier.com/protocol)
- [ZeroTier 文档：Corporate Firewalls](https://docs.zerotier.com/corporate-firewalls)
- [EasyTier 文档：搭建共享节点](https://easytier.cn/guide/network/host-public-server.html)
- [EasyTier 文档：配置文件](https://easytier.cn/guide/network/config-file.html)
- [EasyTier 文档：完整配置项](https://easytier.cn/guide/network/configurations.html)
- [EasyTier 文档：P2P 优化](https://easytier.cn/guide/network/p2p-optimize.html)
- [EasyTier GitHub](https://github.com/EasyTier/EasyTier)
- [GFW Report：How the Great Firewall of China Detects and Blocks Fully Encrypted Traffic](https://gfw.report/publications/usenixsecurity23/zh)


---

> 作者: [枫](https://github.com/qiuzhi)  
> URL: https://blog.iqzhi.com/posts/zerotier-to-easytier-private-networking/  

