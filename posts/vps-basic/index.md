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

### Warp

各大一键脚本，三选一即可。

[FSCARMEN](https://github.com/fscarmen/warp) :

- 首次运行 `wget -N https://raw.githubusercontent.com/fscarmen/warp/main/menu.sh && bash menu.sh`
- 日常维护 `warp`

[P3TERX](https://github.com/P3TERX/warp.sh) :

- 首次运行 `bash <(curl -fsSL git.io/warp.sh) menu`
- 日常维护 `bash warp.sh`

[WARP-GO](https://gitlab.com/ProjectWARP/warp-go/-/tree/master/) :

- 首次运行 `wget -N https://raw.githubusercontent.com/fscarmen/warp/main/warp-go.sh && bash warp-go.sh`
- 日常维护 `warp-go`

### 哪吒监控

到监控管理后台直接拿命令


---

> 作者: [枫](https://github.com/qiuzhi)  
> URL: https://blog.iqzhi.com/posts/vps-basic/  

