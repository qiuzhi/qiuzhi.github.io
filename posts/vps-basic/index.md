# VPS到手基本流程


记录玩VPS的点滴。

<!--more-->

## 简介

记录玩VPS的点滴。

## 系统分支

### Debian

> 适用版本：11（Bullseye）

#### 开启SSH登录

```shell
sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/g' /etc/ssh/sshd_config
systemctl restart ssh
```

#### 更新系统包

```shell
apt update -y && apt upgrade -y
```

#### 安装常用软件

```shell
apt install -y curl
apt install -y socat
```

### IP

```shell
curl -4 ip.sb
curl -6 ip.sb
```

### 网络互联

[ITDOG](https://www.itdog.cn/)

[PING0](http://ip.ping0.cc/)

[ping.pe](https://ping.pe/)

### 融合怪脚本

这个脚本非常不错，虽然是个融合脚本但是有很多别的脚本测不了的东西，有网络信息，IP信息，解锁信息，常用端口开放信息，硬件信息等。关于IP质量问题除了这个以外，IP信息还可以去这里查询，结果非常详细：https://ipinfo.io/

github项目地址：https://github.com/spiritLHLS/ecs

```shell
bash <(wget -qO- --no-check-certificate https://github.com/spiritLHLS/ecs/raw/main/ecs.sh)
bash <(wget -qO- --no-check-certificate https://gitlab.com/spiritysdx/za/-/raw/main/ecs.sh)
```

### 


### 哪吒监控

到监控管理后台直接拿命令

### ZeroTier

安装

```shell
curl -s https://install.zerotier.com/ | bash
```

填写网络ID，加入异地虚拟网络

```shell
zerotier-cli join (网络ID)
```

如果看见`200 join OK`字样就说明成功加入异地虚拟局域网了。

### Warp

各大一键脚本，三选一即可。

[FSCARMEN](https://github.com/fscarmen/warp) :

- 首次运行 
  ```shell
  wget -N https://raw.githubusercontent.com/fscarmen/warp/main/menu.sh && bash menu.sh
  ```
- 日常维护 `warp`

[P3TERX](https://github.com/P3TERX/warp.sh) :

- 首次运行
  ```shell
  bash <(curl -fsSL git.io/warp.sh) menu
  ```
- 日常维护 `bash warp.sh`

[WARP-GO](https://gitlab.com/ProjectWARP/warp-go/-/tree/master/) :

- 首次运行
  ```shell
  wget -N https://raw.githubusercontent.com/fscarmen/warp/main/warp-go.sh && bash warp-go.sh
  ```
- 日常维护 `warp-go`


---

> 作者: [枫](https://github.com/qiuzhi)  
> URL: https://blog.iqzhi.com/posts/vps-basic/  

