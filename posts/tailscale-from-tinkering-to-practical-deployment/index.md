# 从&#34;能连上&#34;到&#34;能管理&#34;：我折腾 Tailscale 的完整过程


一开始，我只想解决一个具体问题：让 Hermes 能通过 SSH 管理宿主机和其他 VPS，同时尽量不把个人 SSH 私钥交给容器里的模型。

这个问题后来牵出了 Tailscale、Docker 网络、Tailscale SSH、子网路由、IP Pool、ZeroTier 和 EasyTier。真正有价值的结论，不是记住&#34;某个参数怎么写&#34;，而是把几个经常混在一起的问题拆开：网络是否可达、由谁发起连接、SSH 如何认证，以及家庭局域网如何安全转发出去。

&lt;!--more--&gt;

## 一 起点：让 Hermes 访问宿主机与 VPS

最初的设想很直接：在宿主机安装 Tailscale，让宿主机和 VPS 加入同一个 Tailnet，然后让 Hermes 容器直接访问远端 VPS 的 Tailscale IP。

这个方案先解决了远端访问这一部分：

```text
Hermes 容器 → 宿主机网络 → Tailscale → 远程 VPS
```

但它并没有自动解决本机访问这一部分：

```text
Hermes 容器 → Hermes 所在宿主机
```

原因在于，Tailscale 身份属于宿主机，而不属于普通 Docker 容器。容器只是借助宿主机出站，并不是独立的 Tailscale 节点。

这里需要把两件事分开：

- **网络可达性**：容器能不能访问目标机器的 TCP 端口；
- **SSH 身份认证**：目标机器如何确认连接者是谁，以及允许登录哪个 Linux 用户。

Tailscale 解决的是第一层；只有启用 Tailscale SSH，它才会进一步接管第二层。

## 二 Tailscale、ZeroTier 和 EasyTier 的职责不同

### 1 Tailscale：身份优先

Tailscale 的优势，是把网络连接和身份管理放在了一起：

```text
用户 / Tag
    ↓
ACL / Grants
    ↓
WireGuard 加密连接
    ↓
Tailscale SSH
```

它基于 WireGuard，提供 NAT 穿透、DERP 中继、MagicDNS、ACL、子网路由和 Tailscale SSH。对 VPS 和基础设施管理来说，这套能力比较完整。

### 2 ZeroTier：虚拟网络优先

ZeroTier 更像一个虚拟交换网络。它的规则、Tag、Capability 和 Flow Rules 都很灵活，在某些二层网络、广播、组播和 mDNS 场景下更友好。

但 ZeroTier 的网络身份不会自动变成 Linux 的 SSH 身份。它可以控制&#34;能否访问目标机器的 22 端口&#34;，却不能替代 SSH 私钥、SSH Agent 或证书。

新版 Central（`central.zerotier.com`）引入了 Organizations 和新的 Flow Rules 模板，管理界面比旧版更现代，但核心协议仍是虚拟二层网络。ZeroTier 的优势没有本质变化，仍然集中在：

```text
适合需要完整二层、广播、组播、mDNS、Device Posture 的场景
```

### 3 EasyTier：自建和国内网络灵活性优先

EasyTier 更适合希望自行控制节点、中继和地址规划的人。它支持 P2P、公共节点、自建节点、子网代理和 OpenWrt/LuCI 相关方案，也可以使用自行规划的虚拟网段。

但它同样没有 Tailscale SSH。EasyTier 能解决&#34;网络如何连接&#34;，不能解决&#34;SSH 如何免传统私钥登录&#34;。

另外，EasyTier 主模式是 TUN，也就是三层 IP 网络；它目前不能原样替代 ZeroTier 的完整二层虚拟以太网能力。像 mDNS 广播、ARP、NetBIOS、AirPrint 自动发现这类依赖二层帧的协议，EasyTier 会缺失。

因此，三者的取舍可以概括为：

```text
Tailscale：身份和管理体验优先
ZeroTier：虚拟二层网络优先
EasyTier：自建、可控和国内链路灵活性优先
```

### 4 VPS 管理优先使用宿主机原生 Tailscale

VPS 上的 Tailscale 有两种安装方式：

| 方式 | 最合适场景 |
| --- | --- |
| 直接命令安装 | 管理 VPS 宿主机、发布子网路由、作为救援通道 |
| Docker 安装 | 让某个容器加入 Tailnet，提供独立服务 |

Docker 版 Tailscale 可以正常运行，但如果目标是：

```text
通过 Tailscale 访问 VPS 宿主机
通过 Tailscale 发布 VPS 后面的网段
```

那么 Docker 方式的限制就会暴露出来：

- 容器重建可能丢失节点身份，必须持久化 `/var/lib/tailscale`；
- 默认 userspace networking 不能完整替代宿主机 TUN 接口；
- Docker 服务异常会导致 Tailscale 通道一起断掉；
- 子网路由、iptables、返回路径比直接安装复杂。

因此，更稳妥的做法是 **在 VPS 宿主机原生安装 `tailscale`**，让 `tailscaled` 作为 systemd 服务运行。Docker 适合只给单个应用分配 Tailnet 身份，不适合把 Tailscale 作为 VPS 管理通道。

### 5 三网共存按区域分工，不按设备铺满

同时使用 Tailscale、EasyTier、ZeroTier 时，三者可以共存，但最容易出问题的地方是：

- 虚拟网段重复；
- 路由抢同一个目标；
- DNS 多次接管冲突；
- 多个 Exit Node 抢默认路由；
- Docker 容器内外看到的接口不一样。

因此，不要把&#34;三套网络都装到每一台设备上&#34;。更合理的做法是按区域分工：

```text
国外 VPS：Tailscale 为主
国内服务器 / 家庭环境：EasyTier 为主
需要二层、广播、组播：ZeroTier 按需启用
Hermes 宿主机 / 管理电脑：同时加入 Tailscale 和 EasyTier
```

并且把三段网在逻辑上彻底分开：

```text
Tailscale：100.64.0.0/10，官方分配，IP Pool 可按 tag 裁剪
EasyTier：10.126.126.0/24（示例）
ZeroTier：10.200.0.0/24（按需）
家庭 LAN：保持现有 192.168.1.0/24
```

**不要**让两个 VPN 同时发布同一个家庭 LAN 网段，也**不要**同时启用多个 0.0.0.0/0 Exit Node。

## 三 第一次验证：不要只凭延迟判断中继

系统自带的 `ping` 只能说明 ICMP 可达，不能说明 Tailscale 当前走的是直连、DERP 还是 Peer Relay。

应当直接使用：

```bash
tailscale ping &lt;目标 Tailscale IP&gt;
```

例如：

```bash
tailscale ping 100.123.36.113
```

一次实际结果如下：

```text
pong from kcskawsm (100.123.36.113) via 198.23.242.105:41641 in 11ms
```

其中：

```text
via 198.23.242.105:41641
```

这说明已经建立 UDP 点对点直连，而不是经过：

```text
DERP(...)
peer-relay(...)
```

`tailscale ping` 显示的 `via 198.23.242.105:41641` 后面是对方公网地址和 WireGuard 端口，说明两端已经 UDP 打洞成功，当前使用的是直连。

第一次普通 `ping` 延迟较高、随后下降，也不能单独作为结论。Tailscale 通常先通过中继完成发现和协商，随后尝试升级为直连。判断路径时，应以 `tailscale ping`，或有持续传输时的 `tailscale status` 为准。

## 四 空闲状态下 `tailscale status` 可能省略 direct

当连接没有持续业务流量时，`tailscale status` 可能显示：

```text
idle, tx ... rx ...
```

这不表示&#34;没有直连&#34;，而是当前连接处于空闲状态，输出省略了实时路径信息。

持续产生流量后再查看：

```bash
ping 100.123.36.113
```

另一个终端执行：

```bash
tailscale status
```

在这种情况下，典型状态会显示：

```text
active; direct 198.23.242.105:41641
```

使用中继时，会显示：

```text
active; relay &#34;xxx&#34;
```

从外部判断连接是否直连时，不要只看 `ping` 延迟。`tailscale ping` 的返回更直接：出现 `via DERP(...)` 或 `peer-relay(...)` 就不是直连；出现 `via &lt;公网IP&gt;:&lt;端口&gt;` 则大概率已经直连。

## 五 通过命令输出判断直连与中继

可以从两个层面判断：

- **ICMP 直连**：`tailscale ping` 输出里出现 `direct`；
- **UDP 打洞成功**：输出里出现 `via &lt;公网IP&gt;:&lt;端口&gt;`。

例如输出：

```text
pong from ... via DERP(japan1:41641) in 25ms
```

这说明走的是 relay 服务器，而不是直连。实际测试中，小米机器到日本 VPS 可以做到 33ms 以内的 P2P；部分国内机器之间由于移动 CGNAT 较重，先经过 DERP、再升级为 P2P，也完全正常。

## 六 Tailscale SSH：固定使用 22 端口，但不会破坏普通 SSH

Tailscale SSH 是这次方案里最关键的功能之一。启用后，目标设备的 Tailscale IP 上的 TCP 22 端口由 Tailscale SSH 接管：

```text
Tailscale IP:22 → Tailscale SSH
```

它使用 Tailscale 节点身份和 Tailnet 策略认证，不要求客户端提供传统 SSH 私钥。

但 Tailscale SSH 有一个限制：它目前固定使用端口 22，不能改成自定义端口。

若系统普通 SSH 已经改成 2222 端口，启用 Tailscale SSH 后可以形成这样的并存关系：

```text
公网 IP:2222          → 普通 OpenSSH
Tailscale IP:2222     → 普通 OpenSSH
Tailscale IP:22       → Tailscale SSH
```

Tailscale SSH 不会修改：

```text
/etc/ssh/sshd_config
~/.ssh/authorized_keys
```

所以公网普通 SSH 仍然可以作为独立的救援通道保留。

目标 VPS 的典型启动命令是：

```bash
sudo tailscale up \
  --hostname=merope \
  --accept-dns=false \
  --ssh
```

这里：

- `--hostname=merope`：设置 Tailscale 设备名称；
- `--accept-dns=false`：不接受 Tailscale 下发的系统 DNS 配置；
- `--ssh`：启用 Tailscale SSH 服务端。

## 七 Hostname、IP 与日常记忆

默认 Tailscale 地址来自：

```text
100.64.0.0/10
```

这个地址空间很大，随机分配出来的地址不适合人工记忆。我把一台 VPS 的名称和地址明确设为：

```text
设备名：ccs-3c4g
Tailscale IP：100.98.76.105
```

Hostname 不能包含空格，适合使用以下形式：

```text
ccs-3c4g
merope
prod-vps-01
```

不适合使用：

```text
广州生产 VPS
my vps
```

机器名称和备注应当分开。Hostname 用于 DNS、SSH 和设备识别；中文说明放到资产清单或备注里。

日常连接优先使用：

```bash
ssh root@ccs-3c4g
```

IP 主要用于排障和应急：

```bash
ssh root@100.98.76.105
```

## 八 Tailnet 地址空间与实际路由范围

Tailscale 默认使用 `/10`，但这不意味着所有 `100.64.0.0/10` 流量都会被强制送进 Tailscale。

在 Linux 上，正常情况下主要是已知 Tailnet 节点对应的 `/32` 路由进入 `tailscale0`。未被使用的地址仍然走原来的默认路由。

在 `ccs-3c4g` 上实际检查过：

```bash
ip route get 100.98.76.1
```

未使用的地址仍走普通网卡和默认网关，而不是 `tailscale0`。因此，移动宽带使用 CGNAT，并不自动意味着 Tailscale 一定冲突。

Tailscale 确实支持 Beta 版 IP Pool，可以把分配范围限制到一个较小的子网，例如：

```text
100.98.76.0/24
```

Policy 中可以配置：

```json
{
  &#34;nodeAttrs&#34;: [
    {
      &#34;target&#34;: [&#34;autogroup:admin&#34;],
      &#34;ipPool&#34;: [&#34;100.98.76.0/24&#34;]
    }
  ]
}
```

这样新加入的管理员设备会从这个池中自动分配地址，不必逐台在后台手工指定。

需要注意：IP Pool 只保证地址属于这个范围，不保证自动分配一定从 `.10`、`.11`、`.12` 顺序开始。已有设备通常也不会自动全部迁移，关键设备才需要在后台或通过 API 单独调整。

## 九 用 IP Pool 与 Tag 规划 VPS 地址

Tailscale 默认从 `100.64.0.0/10` 中随机分配地址。这个范围太大，不方便按用途记忆。官方提供了 IP Pool，可以按 tag 限定新节点的分配范围。

这里有两个关键点：

- IP Pool 目前是 **beta** 功能；
- **安装时不能指定某个具体 IP**，只能通过 `tag` 间接决定&#34;从哪个 /24 里自动拿一个&#34;。

假设想把地址按用途分开：

```text
tag:vps-oversea  →  100.98.76.0/24
tag:vps-cn       →  100.98.77.0/24
tag:home         →  100.98.78.0/24
```

第一步，在 Tailscale 管理后台修改 Access Controls，加上 tagOwners 和 nodeAttrs：

```json
{
  &#34;tagOwners&#34;: {
    &#34;tag:vps-oversea&#34;: [&#34;autogroup:admin&#34;],
    &#34;tag:vps-cn&#34;: [&#34;autogroup:admin&#34;],
    &#34;tag:home&#34;: [&#34;autogroup:admin&#34;]
  },
  &#34;nodeAttrs&#34;: [
    {
      &#34;target&#34;: [&#34;tag:vps-oversea&#34;],
      &#34;ipPool&#34;: [&#34;100.98.76.0/24&#34;]
    },
    {
      &#34;target&#34;: [&#34;tag:vps-cn&#34;],
      &#34;ipPool&#34;: [&#34;100.98.77.0/24&#34;]
    },
    {
      &#34;target&#34;: [&#34;tag:home&#34;],
      &#34;ipPool&#34;: [&#34;100.98.78.0/24&#34;]
    }
  ]
}
```

第二步，为每种用途生成带 tag 的 auth key，而不是复用个人 key：

```text
国外 VPS：tag:vps-oversea
国内 VPS：tag:vps-cn
家庭路由：tag:home
```

生成时勾选 Reusable、指定对应 tag、不要勾 Ephemeral。

第三步，VPS 安装时只传入 auth key 和 hostname：

```bash
curl -fsSL https://tailscale.com/install.sh | sh

sudo tailscale up \
  --auth-key=tskey-auth-xxxx \
  --hostname=overseas-vps-01 \
  --accept-dns=false
```

节点加入后，会自动变成 tagged 设备，并从对应 `/24` 中拿到地址。验证：

```bash
tailscale status
tailscale ip -4
```

若已有一台设备被手工固定为 `100.98.76.105`，后续给它添加同样的 `tag:vps-oversea`，新节点仍会从这个网段分配，不会与它冲突。更关键的是：**不要把同一台机器同时打多个用途 tag**，否则它可能命中多个 ipPool，分配结果反而不稳定。

对于个人电脑、笔记本、手机，不要打 tag。它们走默认分配即可，也便于和其他人账号区分。Tag 只给 VPS、路由器、CI 这类服务设备用。

## 十 最终网络结构：由 KWRT 负责家庭子网转发

家庭网络不需要在每台设备上安装 Tailscale。更合理的做法，是把 KWRT 路由器作为 Subnet Router：

```text
远程 Hermes / VPS
        │
        │ Tailscale
        ▼
KWRT：Tailscale 子网路由器
        │
        │ 家庭 LAN，例如 192.168.1.0/24
        ├── NAS
        ├── 家庭电脑
        ├── 家庭宿主机
        └── 其他局域网设备
```

配置分为以下几个环节：

### 1 在 KWRT 安装 Tailscale

优先使用 KWRT 自己的软件源中的：

```text
tailscale
```

需要 LuCI 图形界面时，再安装：

```text
luci-app-tailscale
```

但 LuCI 插件只是管理层，核心连接仍由 `tailscale` 和 `tailscaled` 提供。

### 2 开启 IP 转发

```bash
sysctl net.ipv4.ip_forward
```

确认结果为：

```text
net.ipv4.ip_forward = 1
```

### 3 发布家庭 LAN

假设家庭 LAN 是：

```text
192.168.1.0/24
```

在 KWRT 执行：

```bash
tailscale set --advertise-routes=192.168.1.0/24
```

### 4 在后台批准路由

进入 Tailscale 管理后台：

```text
Machines → KWRT 设备 → Subnets → Edit
```

批准：

```text
192.168.1.0/24
```

### 5 配置 KWRT 防火墙

允许：

```text
tailscale → lan
```

这表示允许远程 Tailnet 设备访问家庭局域网。初始阶段保留默认 SNAT，不要急着关闭；这样家庭设备不需要额外添加返回 `100.64.0.0/10` 的路由。

### 6 远端客户端接受子网路由

需要访问家庭 LAN 的 Linux 客户端执行：

```bash
sudo tailscale set --accept-routes=true
```

然后测试：

```bash
ping 192.168.1.1
ping 192.168.1.100
```

子网发布、后台批准、ACL 放行、客户端接受路由是四个不同环节，缺少任何一个，都可能出现&#34;看起来配置了但实际不通&#34;的情况。

## 十一 Tailscale 免费计划的适用边界

Tailscale Personal（免费）计划包含：

- 最多 6 个用户；
- 无限用户设备；
- 最多 3 个 ACL groups；
- 最多 50 个 tagged 设备。

这意味着：

- 管理你自己和家人、少量同事，通常够用；
- SOHO / 家庭实验室的 VPS、路由器、NAS 如果都打 tag，要注意 50 台上限；
- 多租户分组能力在免费版里只有 3 个，复杂企业场景会不够。

以我目前的规模：

```text
两三台国外 VPS
两三台国内 VPS
家庭 KWRT
管理电脑
```

Personal 计划足够使用。团队扩大后，再考虑 Standard 或按 tagged resource 付费扩展。

## 十二 EasyTier 接口名称与 Docker 部署注意事项

使用 Docker 版 EasyTier 时，最容易遇到的是两个实际问题：

- 接口名称是什么；
- 如何把接口名称固定下来。

具体情况取决于 EasyTier 是直接安装在宿主机，还是运行在 Docker 中。

直接安装在宿主机时，可以通过参数控制 TUN 接口名：

```bash
--dev-name easytier
```

不指定时由系统决定，通常是 `easytier` 或 `tun`，不同机器之间不保证一致。

Docker 版的情况没有这么直接。容器里的 `easytier` 看到的接口属于容器网络命名空间，宿主机不一定能直接 `ip link` 看到它。除非你使用：

```yaml
network_mode: host
cap_add:
  - NET_ADMIN
  - NET_RAW
devices:
  - /dev/net/tun:/dev/net/tun
```

如果不把 TUN 放到宿主机命名空间，容器重建或迁移后，设备名、PID、接口编号都可能变化。更可靠的做法，是在 EasyTier 配置中显式指定 `dev-name`，并持久化这份配置。

## 十三 最终采用的原则

最后，方案收敛为几条简单原则。

### 1 Tailscale 负责网络，SSH 负责权限

Tailscale 解决设备之间如何安全到达；Tailscale SSH 或普通 OpenSSH 负责登录认证；Linux 用户、sudoers 和强制命令负责限制操作权限。

不能把&#34;能访问 22 端口&#34;等同于&#34;已经完成安全的 SSH 管理&#34;。

### 2 宿主机原生 Tailscale 优先

Hermes 的 Docker 网络先保持不变：

- 不改原有 bridge；
- 不改已发布端口；
- 不急着加入 Sidecar；
- 不在 Hermes 主容器里硬塞一个长期运行的 `tailscaled`。

只要宿主机原生 Tailscale 已经能让容器访问目标 Tailnet IP，就先采用最小改动方案。

### 3 Tailscale SSH 只在真正需要时启用

Tailscale SSH 固定占用 Tailscale IP 的 22 端口，但不影响公网自定义 SSH 端口。公网普通 SSH 仍然保留为救援通道，不要在新方案刚上线时把所有旧通道一起删掉。

### 4 子网路由优先放在路由器上

家庭 LAN 的出口节点应当是 KWRT，而不是 Hermes 容器或某台临时 VPS。这样网络边界清楚，路由器重启后的状态和配置也更容易理解。

### 5 IP 用于规划，Hostname 用于日常使用

IP Pool 可以让地址范围更容易识别，例如：

```text
100.98.76.0/24
```

但日常连接仍然应该使用：

```text
ccs-3c4g
merope
kwrt-home
```

不要把整套基础设施建立在一张需要人工背诵的 IP 表上。

## 十四 结尾：解决的其实不是一个 VPN 问题

表面上，这次是在比较 Tailscale、ZeroTier 和 EasyTier；实际上，是在重新划分基础设施的责任边界：

```text
身份：谁可以连接
网络：连接应该走哪里
路由：哪些子网需要被发布
认证：SSH 如何登录
授权：登录后能做什么
容灾：原来的救援通道是否还在
```

在这套具体需求里，Tailscale 不是每个维度都最好，但最适合作为主网络：

```text
Tailscale 节点：负责身份和加密网络
Tailscale SSH：负责无传统私钥的基础认证
KWRT：负责家庭 LAN 子网转发
Hostname：负责日常记忆
普通公网 SSH：保留为救援通道
```

最终的立场也很明确：国外 VPS 和 Hermes 侧以 Tailscale 为主；国内网络按需使用 EasyTier；只有明确需要二层网络、广播或组播时，才启用 ZeroTier。

最重要的收获是：**不要为了追求一个&#34;完美网络&#34;把所有东西都重构一遍。先验证最小路径，确认真实路由和真实权限，再决定是否引入 Sidecar、代理或第二套 Overlay 网络。**

## 参考

- 🔗 [Tailscale Pricing](https://tailscale.com/pricing)
- 🔗 [Tailscale SSH](https://tailscale.com/docs/features/tailscale-ssh)
- 🔗 [Tailscale 子网路由](https://tailscale.com/docs/features/subnet-routers)
- 🔗 [Tailscale IP Pool](https://tailscale.com/docs/reference/ip-pool)
- 🔗 [Tailscale 连接类型](https://tailscale.com/docs/reference/connection-types)
- 🔗 [Tailscale Docker 配置](https://tailscale.com/docs/features/containers/docker)
- 🔗 [Tailscale Docker 参数](https://tailscale.com/docs/features/containers/docker/docker-params)
- 🔗 [ZeroTier Rules Engine](https://docs.zerotier.com/rules/)
- 🔗 [ZeroTier New Central](https://docs.zerotier.com/new-central/)
- 🔗 [EasyTier GitHub](https://github.com/EasyTier/EasyTier)
- 🔗 [EasyTier 官方文档](https://easytier.cn/en/)
- 🔗 [OpenWrt Tailscale 文档](https://openwrt.org/docs/guide-user/services/vpn/tailscale/start)
- 🔗 [Tailscale Auth Keys](https://tailscale.com/docs/features/access-control/auth-keys)
- 🔗 [Tailscale Tags](https://tailscale.com/docs/features/tags)


---

> 作者: [枫](https://github.com/qiuzhi)  
> URL: https://blog.iqzhi.com/posts/tailscale-from-tinkering-to-practical-deployment/  

