# 从“能连上”到“能管理”：我折腾 Tailscale 的完整过程


一开始，我只是想解决一个很具体的问题：让 Hermes 能够通过 SSH 管理宿主机和其他 VPS，同时尽量不把我的个人 SSH 私钥交给容器里的模型。

结果，这个问题一路牵出了 Tailscale、Docker 网络、Tailscale SSH、子网路由、IP Pool、ZeroTier 和 EasyTier。最后真正有价值的结论不是“某个参数怎么写”，而是把几个经常被混在一起的问题拆开：网络能不能到达、谁在发起连接、SSH 如何认证，以及家庭局域网如何被安全地转发出去。

&lt;!--more--&gt;

## 一 起点：Hermes 到底怎么连接宿主机

最初的设想很直接：在宿主机安装 Tailscale，让宿主机和 VPS 加入同一个 Tailnet，然后让 Hermes 容器直接访问远端 VPS 的 Tailscale IP。

这个方案确实解决了一部分问题：

```text
Hermes 容器 → 宿主机网络 → Tailscale → 远程 VPS
```

但它没有自动解决另一部分：

```text
Hermes 容器 → Hermes 所在宿主机
```

原因在于，Tailscale 身份属于宿主机，不属于普通 Docker 容器。容器只是借助宿主机出站，并不是一个独立的 Tailscale 节点。

这时必须区分两件事：

- **网络可达性**：容器能不能访问目标机器的 TCP 端口；
- **SSH 身份认证**：目标机器如何确认连接者是谁，以及允许登录哪个 Linux 用户。

Tailscale 解决的是第一层；只有启用 Tailscale SSH 时，它才会进一步接管第二层。

## 二 Tailscale、ZeroTier 和 EasyTier，不是完全相同的选择

### 1 Tailscale：身份优先

Tailscale 的优势在于把网络连接和身份管理放在了一起：

```text
用户 / Tag
    ↓
ACL / Grants
    ↓
WireGuard 加密连接
    ↓
Tailscale SSH
```

它基于 WireGuard，默认提供 NAT 穿透、DERP 中继、MagicDNS、ACL、子网路由和 Tailscale SSH。对于管理 VPS 和基础设施来说，整体体验比较完整。

### 2 ZeroTier：虚拟网络优先

ZeroTier 更像一个虚拟交换网络。它的规则、Tag、Capability 和 Flow Rules 很灵活，对某些二层网络、广播、组播和 mDNS 场景更友好。

但 ZeroTier 的网络身份不会自动变成 Linux 的 SSH 身份。它可以控制“能不能访问目标机器的 22 端口”，却不会替代 SSH 私钥、SSH Agent 或证书。

### 3 EasyTier：自建和国内网络灵活性优先

EasyTier 更适合希望自己控制节点、中继和地址规划的人。它支持 P2P、公共节点、自建节点、子网代理和 OpenWrt/LuCI 相关方案，也可以使用自己规划的虚拟网段。

但它同样没有 Tailscale SSH。EasyTier 能解决“网络怎么连”，不能解决“SSH 如何免传统私钥登录”。

因此，三者的取舍可以简单概括为：

```text
Tailscale：身份和管理体验优先
ZeroTier：虚拟二层网络优先
EasyTier：自建、可控和灵活性优先
```

我的实际需求是 Hermes 管理宿主机和 VPS，并希望尽量减少 SSH 私钥暴露，所以最终仍然选择 Tailscale 作为主网络。

## 三 第一次验证：不要凭延迟猜中继

普通的系统 `ping` 只能说明 ICMP 可达，不能说明 Tailscale 当前走的是直连、DERP 还是 Peer Relay。

真正应该使用：

```bash
tailscale ping &lt;目标 Tailscale IP&gt;
```

例如：

```bash
tailscale ping 100.123.36.113
```

一次实际结果是：

```text
pong from kcskawsm (100.123.36.113) via 198.23.242.105:41641 in 11ms
```

其中：

```text
via 198.23.242.105:41641
```

说明已经建立了 UDP 点对点直连，而不是经过：

```text
DERP(...)
peer-relay(...)
```

第一次普通 `ping` 的延迟较高、后续下降，也不能单独作为结论。Tailscale 通常先通过中继完成发现和协商，随后尝试升级为直连。判断路径时，应以 `tailscale ping` 或正在传输时的 `tailscale status` 为准。

## 四 为什么 `tailscale status` 有时不显示 direct

当连接没有持续业务流量时，`tailscale status` 可能显示：

```text
idle, tx ... rx ...
```

这不是“没有直连”，而是当前连接处于空闲状态，输出中省略了实时路径信息。

持续产生流量时再看：

```bash
ping 100.123.36.113
```

另一个终端执行：

```bash
tailscale status
```

典型状态才会显示：

```text
active; direct 198.23.242.105:41641
```

如果是中继，则会显示：

```text
active; relay &#34;xxx&#34;
```

## 五 Tailscale SSH：固定使用 22 端口，但不会破坏普通 SSH

Tailscale SSH 是这次方案里最关键的功能之一。启用后，目标设备的 Tailscale IP 上的 TCP 22 端口由 Tailscale SSH 接管：

```text
Tailscale IP:22 → Tailscale SSH
```

它使用 Tailscale 节点身份和 Tailnet 策略认证，不要求客户端提供传统 SSH 私钥。

但 Tailscale SSH 有一个限制：它目前固定使用端口 22，不能改成自定义端口。

如果系统普通 SSH 已经改成了 2222 端口，启用 Tailscale SSH 后可以形成这样的并存关系：

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

## 六 Hostname、IP 和人类记忆

默认 Tailscale 地址来自：

```text
100.64.0.0/10
```

这个地址空间很大，随机分配出来的地址不适合人工记忆。后来我把一台 VPS 的名称和地址明确改成：

```text
设备名：ccs-3c4g
Tailscale IP：100.98.76.105
```

Hostname 不能包含空格，适合使用：

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

机器名称和备注应该分开。Hostname 用于 DNS、SSH 和设备识别；中文说明放到资产清单或备注里。

日常连接优先使用：

```bash
ssh root@ccs-3c4g
```

IP 主要用于排障和应急：

```bash
ssh root@100.98.76.105
```

## 七 要不要把整个 Tailnet 改成一个小网段

Tailscale 默认使用 `/10`，但并不意味着所有 `100.64.0.0/10` 流量都会被强制送进 Tailscale。

在 Linux 上，正常情况下主要是已知 Tailnet 节点对应的 `/32` 路由进入 `tailscale0`。未被使用的地址仍然走原来的默认路由。

在 `ccs-3c4g` 上实际检查过：

```bash
ip route get 100.98.76.1
```

未使用的地址仍走普通网卡和默认网关，而不是 `tailscale0`。因此，移动宽带使用 CGNAT 的事实并不自动意味着 Tailscale 必然冲突。

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

这样新加入的管理员设备会从这个池中自动分配地址，不需要每台设备都去后台手工指定。

需要注意：IP Pool 只保证地址属于这个范围，不保证自动分配一定从 `.10`、`.11`、`.12` 顺序开始。已有设备通常也不会自动全部迁移，关键设备才需要在后台或通过 API 单独调整。

## 八 最终网络结构：KWRT 负责家庭子网转发

家庭网络并不需要把 Tailscale 安装到每一台设备上。更合理的方式是把 KWRT 路由器作为 Subnet Router：

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

配置过程是：

### 1 在 KWRT 安装 Tailscale

优先使用 KWRT 自己的软件源中的：

```text
tailscale
```

如果需要 LuCI 图形界面，再安装：

```text
luci-app-tailscale
```

但 LuCI 插件只是管理层，核心连接仍然是 `tailscale` 和 `tailscaled`。

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

也就是允许远程 Tailnet 设备访问家庭局域网。初始阶段保留默认 SNAT，不要急着关闭。这样家庭设备不需要额外添加返回 `100.64.0.0/10` 的路由。

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

子网发布、后台批准、ACL 允许、客户端接受路由是四个不同环节，少任何一个都可能“看起来配置了但实际不通”。

## 九 最终采用的原则

折腾到最后，方案收敛成了几条简单原则。

### 1 Tailscale 负责网络，SSH 负责权限

Tailscale 解决设备之间如何安全到达；Tailscale SSH 或普通 OpenSSH 负责登录认证；Linux 用户、sudoers 和强制命令负责限制操作权限。

不能把“能访问 22 端口”误认为“已经完成安全的 SSH 管理”。

### 2 宿主机原生 Tailscale优先

Hermes Docker 网络先保持不变：

- 不改原有 bridge；
- 不改已发布端口；
- 不急着加入 Sidecar；
- 不在 Hermes 主容器里硬塞一个长期运行的 `tailscaled`。

如果宿主机原生 Tailscale 已经能让容器访问目标 Tailnet IP，就先用最小改动方案。

### 3 Tailscale SSH 只在真正需要时启用

Tailscale SSH 固定占用 Tailscale IP 的 22 端口，但不影响公网自定义 SSH 端口。公网普通 SSH 仍然保留作为救援通道，不要在新方案刚上线时把所有旧通道一起删掉。

### 4 子网路由优先放在路由器上

家庭 LAN 的出口节点应该是 KWRT，而不是 Hermes 容器，也不是某个临时 VPS。这样网络边界清楚，路由器重启和配置也更容易理解。

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

## 十 结尾：真正解决的不是一个 VPN 问题

这次折腾表面上是在比较 Tailscale、ZeroTier 和 EasyTier，实际上是在重新划分基础设施的责任边界：

```text
身份：谁可以连接
网络：连接应该走哪里
路由：哪些子网需要被发布
认证：SSH 如何登录
授权：登录后能做什么
容灾：原来的救援通道是否还在
```

Tailscale 最终被选中，不是因为它在每个维度都最好，而是因为它在这套具体需求里最省心：

```text
Tailscale 节点：负责身份和加密网络
Tailscale SSH：负责无传统私钥的基础认证
KWRT：负责家庭 LAN 子网转发
Hostname：负责日常记忆
普通公网 SSH：保留为救援通道
```

最重要的收获是：**不要为了追求一个“完美网络”把所有东西都重构一遍。先验证最小路径，确认真实路由和真实权限，再决定是否引入 Sidecar、代理或第二套 Overlay 网络。**

## 参考

- 🔗 [Tailscale Pricing](https://tailscale.com/pricing)
- 🔗 [Tailscale SSH](https://tailscale.com/docs/features/tailscale-ssh)
- 🔗 [Tailscale 子网路由](https://tailscale.com/docs/features/subnet-routers)
- 🔗 [Tailscale IP Pool](https://tailscale.com/docs/reference/ip-pool)
- 🔗 [Tailscale 连接类型](https://tailscale.com/docs/reference/connection-types)
- 🔗 [ZeroTier Rules Engine](https://docs.zerotier.com/rules/)
- 🔗 [EasyTier GitHub](https://github.com/EasyTier/EasyTier)
- 🔗 [OpenWrt Tailscale 文档](https://openwrt.org/docs/guide-user/services/vpn/tailscale/start)


---

> 作者: [枫](https://github.com/qiuzhi)  
> URL: https://blog.iqzhi.com/posts/tailscale-from-tinkering-to-practical-deployment/  

