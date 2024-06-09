# VPS到手基本流程


记录玩VPS的点滴。

<!--more-->

## 一 简介

记录玩VPS的点滴。

## 二 关键记录

### 1 Debian

> 适用版本：11（Bullseye）、12

#### 1.1 查看内核版本

```bash
uname -r
```

#### 1.2 开启SSH登录

```bash
sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/g' /etc/ssh/sshd_config
systemctl restart ssh
```

#### 1.3 更新系统包

```bash
apt update -y && apt upgrade -y
```

{{< admonition question "`apt-get update失败 Err:1 http://archive.ubuntu.com/ubuntu xenial InRelease`">}}

出现此问题，一般是因为DNS设置的问题，将DNS设置为 *8.8.8.8*

通过下面命令查看DNS

```bash
cat /etc/resolv.conf
```

通过下面命令修改DNS

```bash
echo "nameserver 8.8.8.8" | tee /etc/resolv.conf > /dev/null
```

修改后再次查看

```bash
cat /etc/resolv.conf
nameserver 8.8.8.8
```

说明设置成功。

{{< /admonition >}}

#### 1.4 安装常用软件

```bash
apt install -y sudo
apt install -y curl
apt install -y socat
```

#### 1.5 创建新用户并允许远程SSH远程登录，并禁止root用户远程SSH登录

1. 在 Debian 中添加 sudo 用户

首先，要创建用户，当前用户必须是 root 用户或者 sudo 用户。

使用下面adduser 命令创建一个用户名为test的sudo用户，按照提示输入密码，使用 adduser 命令，还会创建用户的主目录。

```bash
adduser test
```

2. 将用户设为 sudo 用户

创建test用户后，可以使用 -aG 组合选项将其添加到 sudo 组，就可以将其转为 sudo 用户。使用 -a 选项是为了确保向组中“追加”。

```bash
usermod -aG sudo test
```

3. 验证 sudo 权限

使用下面命令验证test用户是否被赋予sudo权限，在命令输出中，末尾你会看到是否可以 sudo 权限运行所有命令：(ALL : ALL) ALL

```bash
sudo -l -U test
```

4. 修改 `/etc/ssh/sshd_config` 文件

将 `PermitRootLogin` 设置为 `no` ，`PasswordAuthentication` 设置为 `yes` 即可，保存退出即可。

5. 重启 SSH 服务

```bash
systemctl start ssh.service
/etc/init.d/ssh restart
```

### 4 系统通用

### 4.1 IP

```bash
curl -4 ip.sb
curl -6 ip.sb
```

### 4.2 网络互联

[ITDOG](https://www.itdog.cn/)

[PING0](http://ip.ping0.cc/)

[ping.pe](https://ping.pe/)

### 4.3 融合怪脚本

这个脚本非常不错，虽然是个融合脚本但是有很多别的脚本测不了的东西，有网络信息，IP信息，解锁信息，常用端口开放信息，硬件信息等。关于IP质量问题除了这个以外，IP信息还可以去这里查询，结果非常详细：https://ipinfo.io/

github项目地址：https://github.com/spiritLHLS/ecs

```bash
bash <(wget -qO- --no-check-certificate https://github.com/spiritLHLS/ecs/raw/main/ecs.sh)
bash <(wget -qO- --no-check-certificate https://gitlab.com/spiritysdx/za/-/raw/main/ecs.sh)
```

### 4.4 添加 SWAP

swap 是 Linux 中的虚拟内存，用于扩充物理内存不足而用来存储临时数据存在的。它类似于 Windows 中的虚拟内存。在 Windows 中，只可以使用文件来当作虚拟内存。而 linux 可以文件或者分区来当作虚拟内存。

这个虚拟内存对于内存小的 VPS 非常有必要，可以提高我们的运行效率。`建议设置为实际ram的 2 倍。`

```bash
wget -O box.sh https://raw.githubusercontent.com/BlueSkyXN/SKY-BOX/main/box.sh && chmod +x box.sh && clear && ./box.sh
```

### 4.5 哪吒监控

从自建部署的监控管理后台直接拿命令。

### 4.6 Docker

官方一键安装脚本

```bash
wget -qO- get.docker.com | bash
#查看docker版本
docker -v
```

Docker-compose 安装

```bash
curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-Linux-x86_64 > /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
docker-compose --version
```

#### 4.6.1 修改 Docker 配置

以下配置会增加一段自定义内网 IPv6 地址，开启容器的 IPv6 功能，以及限制日志文件大小，防止 Docker 日志塞满硬盘（泪的教训）：

```bash
cat > /etc/docker/daemon.json <<EOF
{
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "20m",
        "max-file": "3"
    },
    "ipv6": true,
    "fixed-cidr-v6": "fd00:dead:beef:c0::/80",
    "experimental":true,
    "ip6tables":true
}
EOF
```

然后重启 Docker 服务：

```bash
systemctl restart docker
```

好了，我们已经安装好了 Docker 和 Docker Compose，然后就可以开始愉快的安装各种软件。

### 4.7 ZeroTier

安装

```bash
curl -s https://install.zerotier.com/ | bash
```

填写网络ID，加入异地虚拟网络

```bash
zerotier-cli join (网络ID)
```

如果看见`200 join OK`字样就说明成功加入异地虚拟局域网了。

### 4.8 Warp

各大一键脚本，三选一即可。

[FSCARMEN](https://github.com/fscarmen/warp) :

- 首次运行 
  ```bash
  wget -N https://raw.githubusercontent.com/fscarmen/warp/main/menu.sh && bash menu.sh
  ```
- 日常维护 `warp`

[P3TERX](https://github.com/P3TERX/warp.sh) :

- 首次运行
  ```bash
  bash <(curl -fsSL git.io/warp.sh) menu
  ```
- 日常维护 `bash warp.sh`

[WARP-GO](https://gitlab.com/ProjectWARP/warp-go/-/tree/master/) :

- 首次运行
  ```bash
  wget -N https://raw.githubusercontent.com/fscarmen/warp/main/warp-go.sh && bash warp-go.sh
  ```
- 日常维护 `warp-go`


---

> 作者: [枫](https://github.com/qiuzhi)  
> URL: https://blog.iqzhi.com/posts/vps-basic/  

