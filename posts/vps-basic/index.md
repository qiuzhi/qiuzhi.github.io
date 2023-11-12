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

### 哪吒监控

到监控管理后台直接拿命令


---

> 作者: [枫](https://github.com/qiuzhi)  
> URL: https://blog.iqzhi.com/posts/vps-basic/  

