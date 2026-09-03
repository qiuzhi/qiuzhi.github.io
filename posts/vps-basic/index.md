# VPS到手基本流程


记录玩VPS的点滴。

&lt;!--more--&gt;

## 一 简介

记录玩VPS的点滴。

## 二 关键记录

### 1 系统

#### 1.1 重装（DD脚本）

```bash
curl -O https://raw.githubusercontent.com/bin456789/reinstall/main/reinstall.sh
sudo chmod &#43;x ./reinstall.sh
sudo ./reinstall.sh debian 12
```

或者

```bash
wget --no-check-certificate -qO InstallNET.sh &#39;https://raw.githubusercontent.com/leitbogioro/Tools/master/Linux_reinstall/InstallNET.sh&#39; &amp;&amp; chmod a&#43;x InstallNET.sh &amp;&amp; bash InstallNET.sh -debian 12 -pwd &#39;1qaz@WSX&#39;
```

### 3 Debian

&gt; 适用版本：11（Bullseye）、12

#### 3.1 修改语言设置成英文

打开以下文件：

```bash
vi /etc/default/locale
```

将对应内容修改为：

```txt
LANG=&#34;en_US.UTF-8&#34;
LANGUAGE=&#34;en_US:en&#34;
```

更新：

```bash
apt update
```

然后重启reboot机器。

#### 3.2 查看内核版本

```bash
uname -r
```

#### 3.3 开启SSH登录

```bash
sed -i &#39;s/#PermitRootLogin prohibit-password/PermitRootLogin yes/g&#39; /etc/ssh/sshd_config
systemctl restart ssh
```

#### 3.4 更新系统包

```bash
apt update -y &amp;&amp; apt upgrade -y
```

{{&lt; admonition question &#34;`apt-get update失败 Err:1 http://archive.ubuntu.com/ubuntu xenial InRelease`&#34;&gt;}}

出现此问题，一般是因为DNS设置的问题，将DNS设置为 *8.8.8.8*

通过下面命令查看DNS

```bash
cat /etc/resolv.conf
```

通过下面命令修改DNS

```bash
echo &#34;nameserver 8.8.8.8&#34; | tee /etc/resolv.conf &gt; /dev/null
```

修改后再次查看

```bash
cat /etc/resolv.conf
nameserver 8.8.8.8
```

说明设置成功。

{{&lt; /admonition &gt;}}

#### 3.5 更改主机名

备份当前配置

```bash
cp /etc/hostname /etc/hostname.bak
cp /etc/hosts /etc/hosts.bak
```

查看当前主机名

```bash
hostnamectl
```

修改当前主机名

```bash
hostnamectl set-hostname mydebian
```

打开`/etc/hosts`文件，将原主机名修改为新主机名即可。

应用更改

```bash
systemctl restart systemd-hostnamed
```

#### 3.6 时区时间

1. 第三方云主机时区就未必符合本地要求。

```bash
## 查看时区，有 CST 正确
date
## 设置
timedatectl set-timezone Asia/Shanghai
## 或者使用向导选择
tzselect
```

2. 检查授时同步是否打开

检查`ntpd`的状态：

```bash
systemctl status ntp
```

一般来说，我们只作为客户端使用，而不需要作为服务端提供的话，不需要用到`ntp`，直接使用`timesyncd`即可

检查`timesyncd`的状态：

```bash
systemctl status systemd-timesyncd
```

`ntpd`与`timesyncd`冲突，若已安装`ntpd`，需要先删除：

```bash
apt purge ntp
```

安装`timesyncd`：

```bash
apt install -y systemd-timesyncd
```

启动`timesyncd`服务：

```bash
systemctl start systemd-timesyncd
```

输出确认`timesyncd`正在运行。要显示当前时间和日期，请运行：

```bash
timedatectl
```

#### 3.7 安装常用软件

```bash
apt install -y sudo
apt install -y curl
apt install -y socat
```

#### 3.8 创建新用户并允许远程SSH远程登录，并禁止root用户远程SSH登录

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

#### 3.9 查看系统端口状态

```bash
sudo netstat -tulnp | grep &lt;port_number&gt;
```

或者

```bash
sudo ss -tulw | grep &lt;port_number&gt;
```

#### 3.10 BBR

##### 3.10.1 如何检查您的系统是否启用了 BBR？

在启用 BBR 之前，检查它是否已在您的系统上启用是必不可少的。为此，请运行以下命令：

```bash
sysctl net.ipv4.tcp_congestion_control
```

如果启用了 BBR，您将看到以下输出：

```bash
net.ipv4.tcp_congestion_control = bbr
```

如果您看到不同的拥塞控制算法，例如 cubic 或 reno，则 BBR 未启用。

##### 3.10.2 如何在 Debian Linux 中启用 BBR？

要在 Ubuntu Linux 上启用 BBR，请执行以下步骤：

第 1 步：更新您的系统
在对系统进行任何更改之前，更新它以确保您拥有最新的软件包和安全修复程序至关重要。为此，请运行以下命令：

```bash
sudo apt update &amp;&amp; sudo apt-get upgrade
```

第 2 步：检查是否支持 BBR
并非所有系统都支持 BBR，因此检查您的系统是否必不可少。为此，请运行以下命令：

```bash
sudo modprobe tcp_bbr
```

如果您的系统支持 BBR，您将看不到任何输出。如果您的系统不支持 BBR，您将看到一条错误消息。

第 3 步：启用 BBR
要启用 BBR，请运行以下命令：

```bash
sudo sh -c &#39;echo &#34;net.core.default_qdisc=fq&#34; &gt;&gt; /etc/sysctl.conf&#39;
sudo sh -c &#39;echo &#34;net.ipv4.tcp_congestion_control=bbr&#34; &gt;&gt; /etc/sysctl.conf&#39;
```

这些命令会将默认排队规则设置为 fq 并启用 BBR 作为拥塞控制算法。

第 4 步：重新加载 sysctl
要应用更改，请运行以下命令：

```bash
sudo sysctl -p
```

##### 3.10.3 如何验证是否启用了 BBR？

要验证 BBR 是否已启用，请运行以下命令：

```bash
sysctl net.ipv4.tcp_congestion_control
```

如果启用了 BBR，您将看到以下输出：

```bash
net.ipv4.tcp_congestion_control = bbr
```

#### 3.11 swap分区

两种方式：`file` 与 `partition`

#### 3.11.1 swap file（交换文件）

```bash
sudo bash -c &#39;
set -e

swapoff /swapfile 2&gt;/dev/null || true
rm -f /swapfile

fallocate -l 4G /swapfile || dd if=/dev/zero of=/swapfile bs=1M count=4096
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile

grep -q &#34;^/swapfile &#34; /etc/fstab || echo &#34;/swapfile none swap sw 0 0&#34; &gt;&gt; /etc/fstab

sysctl vm.swappiness=70
echo &#34;vm.swappiness=70&#34; &gt; /etc/sysctl.d/99-swap.conf

free -h
&#39;
```

### 4 系统通用

#### 4.1 IP

```bash
curl -4 ip.sb
curl -6 ip.sb
```

#### 4.2 网络互联

[ITDOG](https://www.itdog.cn/)

[PING0](http://ip.ping0.cc/)

[ping.pe](https://ping.pe/)

**原生IP查询**：[iplark](https://iplark.com/)

**在线IP信息查询工具**：[zerosla](https://ip.zerosla.net)

IP质量体检报告：

```bash
bash &lt;(curl -Ls IP.Check.Place)
```

#### 4.3 测试脚本

本项目本质上是测试工具集合的前置加载器和结果后处理项目。把服务器测试工作的流程给规范化自动化了。 让测试仅仅是测试，不要留下一堆痕迹；让测试可以更舒服省心，自动排版截图。

github项目地址：https://github.com/LloydAsp/NodeQuality

```bash
bash &lt;(curl -sL https://run.NodeQuality.com)
```

#### 4.4 添加 SWAP

swap 是 Linux 中的虚拟内存，用于扩充物理内存不足而用来存储临时数据存在的。它类似于 Windows 中的虚拟内存。在 Windows 中，只可以使用文件来当作虚拟内存。而 linux 可以文件或者分区来当作虚拟内存。

这个虚拟内存对于内存小的 VPS 非常有必要，可以提高我们的运行效率。`建议设置为实际ram的 2 倍。`

```bash
wget -O box.sh https://raw.githubusercontent.com/BlueSkyXN/SKY-BOX/main/box.sh &amp;&amp; chmod &#43;x box.sh &amp;&amp; clear &amp;&amp; ./box.sh
```

#### 4.5 哪吒监控

从自建部署的监控管理后台直接拿命令。

#### 4.6 UFW防火墙

```bash
apt install ufw
ufw allow ssh
ufw enable
```

#### 4.7 Docker

官方一键安装脚本

```bash
wget -qO- get.docker.com | bash
#查看docker版本
docker -v
```

国内机安装

```bash
#https://linuxmirrors.cn/other/
bash &lt;(curl -sSL https://linuxmirrors.cn/docker.sh)
#使用阿里云镜像安装
curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun
#使用 Azure 中国镜像安装
curl -fsSL https://get.docker.com | bash -s docker --mirror AzureChinaCloud
```

Docker-compose 安装

```bash
curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-Linux-x86_64 &gt; /usr/local/bin/docker-compose
curl -L https://github.com/docker/compose/releases/latest/download/docker-compose-Linux-aarch64 &gt; /usr/local/bin/docker-compose
sudo chmod &#43;x /usr/local/bin/docker-compose
docker-compose --version
```

##### 4.7.1 修改 Docker 配置

以下配置会增加一段自定义内网 IPv6 地址，开启容器的 IPv6 功能，以及限制日志文件大小，防止 Docker 日志塞满硬盘（泪的教训）：

```bash
cat &gt; /etc/docker/daemon.json &lt;&lt;EOF
{
    &#34;log-driver&#34;: &#34;json-file&#34;,
    &#34;log-opts&#34;: {
        &#34;max-size&#34;: &#34;20m&#34;,
        &#34;max-file&#34;: &#34;3&#34;
    },
    &#34;ipv6&#34;: true,
    &#34;fixed-cidr-v6&#34;: &#34;fd00:dead:beef:c0::/80&#34;,
    &#34;experimental&#34;:true,
    &#34;ip6tables&#34;:true
}
EOF
```

然后重启 Docker 服务：

```bash
systemctl restart docker
```

好了，我们已经安装好了 Docker 和 Docker Compose，然后就可以开始愉快的安装各种软件。

##### 4.7.2 获取 Docker 容器的 IP 地址

查询单个容器 IP 地址：

使用下面命令可以查看容器详细信息，里面包含 IP 地址信息：

```bash
docker inspect &lt;container id&gt;
```

或者使用下面命令直接输出 IP 地址信息：

```bash
docker inspect --format &#39;{{ .NetworkSettings.IPAddress }}&#39; &lt;container id&gt;
```

或者：

```bash
docker inspect -f &#39;{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}&#39; &lt;container id&gt;
```

查询全部容器 IP 地址：

下面三个命令，任选其一即可：

```bash
docker inspect -f &#39;{{.Name}} - {{.NetworkSettings.IPAddress }}&#39; $(docker ps -aq)
```

或者：

```bash
docker inspect -f &#39;{{.Name}} - {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}&#39; $(docker ps -aq)
```

或者：

```bash
docker inspect --format=&#39;{{.Name}} - {{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}&#39; $(docker ps -aq)
```

##### 4.7.3 进入 Docker 容器内部

```bash
docker exec -it &lt;container id&gt; /bin/bash
```

#### 4.8 ZeroTier

安装

```bash
curl -s https://install.zerotier.com/ | bash
```

填写网络ID，加入异地虚拟网络

```bash
zerotier-cli join (网络ID)
```

如果看见`200 join OK`字样就说明成功加入异地虚拟局域网了。

#### 4.9 Warp

各大一键脚本，自选一即可。

FSCARMEN :
 - 首次运行 wget -N https://gitlab.com/fscarmen/warp/-/raw/main/menu.sh &amp;&amp; bash menu.sh
 - 日常维护 warp

P3TERX :
 - 首次运行 bash &lt;(curl -fsSL git.io/warp.sh) menu
 - 日常维护 bash warp.sh

WARP-GO :
 - 首次运行 wget -N https://raw.githubusercontent.com/fscarmen/warp/main/warp-go.sh &amp;&amp; bash warp-go.sh
 - 日常维护 warp-go

MISAKA :
 - 首次运行 wget -N https://gitlab.com/Misaka-blog/warp-script/-/raw/main/warp.sh &amp;&amp; bash warp.sh
 - 日常维护 bash warp.sh

&gt; **开启效果**
&gt;
&gt; 无 - IPv4 不支持 | IPv6 支持 | ~1Gbps | 无解锁
&gt;
&gt; IPv4 Only &#43; IPv6 优先(默认) - IPv4 支持 | IPv6 支持 | 优先 IPv6 (~1Gbps), 否则 IPv4 (~200Mbps) | 无解锁&lt;br&gt;
&gt; IPv4 Only &#43; IPv4 优先 - IPv4 支持 | IPv6 支持 | 优先 IPv4 (~200Mbps), 否则 IPv6 (~1Gbps) | WARP解锁
&gt;
&gt; 双栈 &#43; IPv6 优先(默认) - IPv4 支持 | IPv6 支持 | ~200Mbps | WARP解锁 (部分)&lt;br&gt;
&gt; 双栈 &#43; IPv4 优先(默认) - IPv4 支持 | IPv6 支持 | ~200Mbps | WARP解锁

#### 4.10 打包迁移

只打包 `~/container` 目录下 `hermes` 文件夹

```bash
tar -C ~/container -zcvf hermes-data-$(date &#43;%F).tar.gz hermes
```

解压到当前目录 `hermes` 文件夹

```bash
tar -zxvf hermes-deploy-YYYY-MM-DD.tar.gz
```


## 参考

1. [在Dedian系统上设置时间同步--解决Debian系统时间无法同步](https://cnboy.org/2883)
1. [如何在 Debian 12/11/10 中启用 BBR](https://jigutech.com/2172.html)


---

> 作者: [枫](https://github.com/qiuzhi)  
> URL: https://blog.iqzhi.com/posts/vps-basic/  

