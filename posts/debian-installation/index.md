# 自装Debian系统折腾手记


CentOS离场，把玩debian。

<!--more-->

> 适用版本：11（Bullseye）

```bash
root@debian:~## uname -a
Linux debian 5.10.0-21-amd64 #1 SMP Debian 5.10.162-1 (2023-01-21) x86_64 GNU/Linux
```

## 简言

说起为什么我要折腾Debian，还是因为CentOS不再更新了，在外新买的VPS也都使用了Debian系统，所以也想将目前的服务往Debian转移。

## 安装

从官网下载ISO，加载启动即可。

Ps. 安装时root账号可设置空密码，这样就不能通过SSH登录了，必须另行设置新账号强密码即可。

可选清华的源，在安装过程中会将软件更至最新。

### 时区

我们默认安装是已经正确设置时区，但是如果是第三方云主机时区就未必符合本地要求。

```bash
## 查看时区，有 CST 正确
date
## 设置
sudo timedatectl set-timezone Asia/Shanghai
## 或者使用向导选择
tzselect
```

### IPv6

编辑interfaces文件`/etc/network/interfaces`，在文本最后面添加：

```plaintext
iface xxx inet6 dhcp
```

上面的xxx更换成接口名。

查看ipv6的网关：

```bash
ip -6 route show
```

如果有：**default via fe80::2e2:69ff:fe55:83a4 dev ens192 metric 1 pref medium**，类似这样的出现，说明ipv6一切正常。否则需要手动编辑interfaces文件`/etc/network/interfaces`，在文本最后面添加：

```plaintext
up route -A inet6 add default gw fe80::2e2:69ff:fe55:83a4 dev ens192
```

### zsh

好用的zsh怎么少得了。

```bash
# 查看当前shell
echo $SHELL
# 安装
apt install zsh
# 为 root 设置默认 shell
chsh -s /bin/zsh
```

#### 全局配置 zsh

> 注意：以下全局配置相关命令需要 root 权限，请切换到 root 账号，或者使用 sudo。

全局安装 zsh 到 /etc 目录

```
git clone --depth=1 https://github.com/ohmyzsh/ohmyzsh.git /etc/oh-my-zsh
```

从模板文件复制 .zshrc 创建默认配置文件（新用户将使用该配置文件）

```bash
cp /etc/oh-my-zsh/templates/zshrc.zsh-template /etc/skel/.zshrc
```

修改 on-my-zsh 的安装目录 `export ZSH=$HOME/.oh-my-zsh` 为 `export ZSH=/etc/oh-my-zsh`

```bash
sed -i 's|$HOME/.oh-my-zsh|/etc/oh-my-zsh|g' /etc/skel/.zshrc
```

为每个用户配置独立的 cache 目录

编辑 `/etc/skel/.zshrc` 在 `export ZSH=/etc/oh-my-zsh` 下添加一句：

```bash
export ZSH_CACHE_DIR="${XDG_CACHE_HOME:-$HOME/.cache}/oh-my-zsh"
```

更改默认主题（推荐 ys）

编辑 /etc/skel/.zshrc 文件修改：

```bash
sed -i '/^ZSH_THEME=.*/c ZSH_THEME="ys"' /etc/skel/.zshrc
```

取消每周自动检查更新（新版不用管）

配置 ll 别名（可选）

```bash
echo 'alias ll="ls -lahF --color --time-style=long-iso"' >> /etc/skel/.zshrc
```

#### 全局配置插件

全局安装插件（安装到 /etc/oh-my-zsh/custom/plugins/）

##### [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting)：语法高亮插件

作用：命令错误会显示红色，直到你输入正确才会变绿色，另外路径正确会显示下划线。

```bash
git clone --depth=1 https://github.com/zsh-users/zsh-syntax-highlighting.git /etc/oh-my-zsh/custom/plugins/zsh-syntax-highlighting
```

##### [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)

作用：根据历史输入命令的记录即时的提示（建议补全），然后按 → 键即可补全。

```bash
git clone --depth=1 https://github.com/zsh-users/zsh-autosuggestions.git /etc/oh-my-zsh/custom/plugins/zsh-autosuggestions
```

启用插件

编辑 `/etc/skel/.zshrc`，找到 plugins=(git) 这一行，修改为：

```bash
plugins=([plugins...]zsh-syntax-highlighting zsh-autosuggestions)
```

快速修改：

```bash
sed -i '/^plugins=.*/c plugins=(git zsh-syntax-highlighting zsh-autosuggestions)' /etc/skel/.zshrc
```

#### 使用用户配置文件

改变新用户的默认 shell

```bash
vi /etc/default/useradd
```

将 SHELL= \* (比如 SHELL=/bin/sh) 改成 SHELL=/bin/zsh

```bash
sed -i '/^SHELL=.*/c SHELL=/bin/zsh' /etc/default/useradd
```

修改后，使用 `useradd` 命令无需 `-s /bin/zsh`，用户默认使用 zsh，当然也可以不修改此项，`useradd` 命令继续追加 `-s /bin/zsh` 参数。

新用户登录后，将自动复制 .zshrc 和上述 cache 目录到用户主目录下，并自动加载 zsh 配置。

针对现有用户

直接复制 `/etc/skel/.zshrc` 到 `~/`

```bash
cp /etc/skel/.zshrc ~/.zshrc
source ~/.zshrc
```

### Git

#### 解决中文乱码

```bash
git config --global core.quotepath false
```

### Docker

以下操作需要在 root 用户下完成，请使用 `sudo -i` 或 `su root` 切换到 root 用户进行操作。

首先，安装一些必要的软件包：

```bash
apt update
apt upgrade -y
apt install curl vim wget gnupg apt-transport-https lsb-release ca-certificate
```

然后加入 Docker 的 GPG 公钥和 apt 源：

```bash
wget -O /usr/share/keyrings/docker.asc https://download.docker.com/linux/debian/gpg
echo "deb [signed-by=/usr/share/keyrings/docker.asc] https://download.docker.com/linux/debian $(lsb_release -sc) stable" > /etc/apt/sources.list.d/docker.list
```

国内机器可以用清华 TUNA的国内源：

```bash
wget -O /usr/share/keyrings/docker.asc https://download.docker.com/linux/debian/gpg
echo "deb [signed-by=/usr/share/keyrings/docker.asc] https://mirrors.tuna.tsinghua.edu.cn/docker-ce/linux/debian $(lsb_release -sc) stable" > /etc/apt/sources.list.d/docker.list
```

然后更新系统后即可安装 Docker CE：

```bash
apt update
apt-get install docker-ce docker-ce-cli containerd.io
```

此时可以使用 `docker version` 命令检查是否安装成功：

```bash
root@debian:~## docker version
Client: Docker Engine - Community
 Version:           23.0.1
 API version:       1.42
 Go version:        go1.19.5
 Git commit:        a5ee5b1
 Built:             Thu Feb  9 19:46:54 2023
 OS/Arch:           linux/amd64
 Context:           default

Server: Docker Engine - Community
 Engine:
  Version:          23.0.1
  API version:      1.42 (minimum version 1.12)
  Go version:       go1.19.5
  Git commit:       bc3805a
  Built:            Thu Feb  9 19:46:54 2023
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.6.18
  GitCommit:        2456e983eb9e37e47538f59ea18f2043c9a73640
 runc:
  Version:          1.1.4
  GitCommit:        v1.1.4-0-g5fd4c4d
 docker-init:
  Version:          0.19.0
  GitCommit:        de40ad0
```

#### 常用命令

```bash
# 跟踪日志
docker logs -f --tail=200 xxx

# 删除未使用的镜像
docker image prune
```

#### Docker Compose

我们可以使用 Docker 官方发布的 Github 直接安装最新版本：

```bash
curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-Linux-x86_64 > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

此时可以使用 `docker-compose version` 命令检查是否安装成功：

```bash
root@debian ~ # docker-compose version
Docker Compose version v2.16.0
```

#### 修改 Docker 配置

以下配置会增加一段自定义内网 IPv6 地址，开启容器的 IPv6 功能，以及限制日志文件大小，防止 Docker 日志塞满硬盘（泪的教训）：

```bash
cat > /etc/docker/daemon.json
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
```

然后重启 Docker 服务：

```bash
systemctl restart docker
```

好了，我们已经安装好了 Docker 和 Docker Compose，然后就可以开始愉快的安装各种软件。

### scp 上传下载文件

**1、上传本地文件到服务器**

```bash
scp /path/filename username@servername:/path/
```

**2、从服务器上下载文件**

```bash
scp username@servername:/path/filename /var/www/local\_dir(本地目录)
```

**3、从服务器下载整个目录**

```bash
scp -r username@servername:/var/www/remote\_dir/(远程目录) /var/www/local\_dir(本地目录)
```

**4、上传目录到服务器**

```bash
scp -r local\_dir username@servername:remote\_dir
```

## 挂载

```bash
# 只可以查看已经挂载的分区和文件系统类型。
df -T
# 可以显示出所有挂载和未挂载的分区，但不显示文件系统类型。
fdisk -l
# 也可以查看未挂载的文件系统类型。
lsblk -f
# 可以查看未挂载的文件系统类型，以及哪些分区尚未格式化。
parted -l

# 开机自动挂载
vi /etc/fstab
```

### 硬盘

```bash
# 在 fstab 文件末位增加一行，用于设置开机自动挂载 硬盘
UUID=25b53c64-94ec-4bd6-a322-a1b776ad86c1 /mnt/hdd1 ext4 errors=remount-ro 0 2
```

### cifs

```bash
# 单次挂载
mount -t cifs //192.168.9.23/Disk_sataa5 /mnt/zidoo -o username=xxx,password=xxx
mount -t cifs //192.168.9.23/Disk_sataa5 /mnt/zidoo -o username=guest,password=guest

# 在 fstab 文件末位增加一行，用于设置开机自动挂载 smb
//192.168.9.23/Disk_sataa5 /mnt/zidoo cifs uid=0,gid=0,username=xxx,password=xxx,sec=ntlm 0 2
//192.168.9.23/Disk_sataa5 /mnt/zidoo cifs defaults 0 2

# 若报错可尝试先安装cifs工具包
apt install cifs-utils
```

参考

1. [Debian 11 “bullseye” 安装笔记](https://zhuanlan.zhihu.com/p/402960046?utm_id=0)
2. [Linux 全局安装配置 zsh + oh-my-zsh](https://sysin.org/blog/linux-zsh-all/)
3. [非桌面版Debian 11自动配置获取ipv6地址教程](https://www.mumupc.com/archives/20226.html)
4. [在git中出现中文乱码的解决方案](https://blog.csdn.net/tyro_java/article/details/53439537)


---

> 作者:   
> URL: https://blog.iqzhi.com/posts/debian-installation/  

