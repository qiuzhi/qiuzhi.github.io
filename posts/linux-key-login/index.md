# Linux设置密钥登陆


网上看了几篇对于Linux使用密钥登录的好文章，这里合并做个备用，如有系统安全类需求，这是一个很好的参考思路。

<!--more-->

## 方案一：传统做法

转载：[【有序集合】Linux设置密钥登陆](https://zset.cc/archives/25/)

首先在家目录下创建 `authorized_keys` 用于存放公钥，如果已有该文件可跳过

```bash
mkdir .ssh
touch .ssh/authorized_keys
```

在 `authorized_keys` 文件中添加公钥，并修改文件权限

```bash
chmod 600 .ssh/authorized_keys
chmod 700 .ssh
```

编辑 `/etc/ssh/sshd_config` 文件，进行如下设置：

```config
RSAAuthentication yes
PubkeyAuthentication yes
```

另外，请留意 root 用户能否通过 SSH 登录：

```config
PermitRootLogin yes
```

当你完成全部设置，并以密钥方式登录成功后，再禁用密码登录：

```config
PasswordAuthentication no
```

最后，重启 SSH 服务：

```bash
service sshd restart
```

## 方案二：一键配置脚本

转载：[【P3TERX ZONE】SSH 密钥一键配置脚本 使用教程](https://p3terx.com/archives/ssh-key-installer.html)

对于新入手或重装后的 VPS 配置密钥登录需要创建 `~/.ssh` 目录、把公钥写入到 `~/.ssh/authorized_keys`、设置权限、禁用密码登录等操作，虽然都是很简单的基础操作，但过程麻烦且枯燥：

```bash
mkdir -p ~/.ssh
curl -fsSL https://github.com/P3TERX.keys >> ~/.ssh/authorized_keys
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
sudo sed -i "s@.*\(PasswordAuthentication \).*@\1no@" /etc/ssh/sshd_config
sudo service sshd restart
```

SSH 密钥一键配置脚本是一套用于简化 SSH 密钥配置过程的解决方案。使用它以上操作只需要一行命令：

```bash
bash <(curl -fsSL git.io/key.sh) -g P3TERX -d
```

### 语法及选项说明

```bash
bash <(curl -fsSL git.io/key.sh) [选项...] <参数>
```

- -o - 覆盖模式，必须写在最前面才会生效
- -g - 从 GitHub 获取公钥，参数为 GitHub 用户名
- -u - 从 URL 获取公钥，参数为 URL
- -f - 从本地文件获取公钥，参数为本地文件路径
- -p - 修改 SSH 端口，参数为端口号
- -d - 禁用密码登录

### 使用方法

#### 生成 SSH 密钥对
如果没有密钥需要先生成，执行以下命令后一路回车即可。

```bash
ssh-keygen -t ecdsa -b 521
```

>> TIPS： 此方法适用于 Win­dows 10 (1803后)的 Pow­er­Shell 或 WSL，Linux 发行版和 ma­cOS 自带的终端，但不仅限于这些环境。
>> 科普： 521 位的 ECDSA 密钥比起 RSA 密钥更安全且验证速度更快。

操作完后会在 `~/.ssh` 目录中生两个密钥文件，`id_ecdsa` 为私钥，`id_ecdsa.pub` 为公钥。公钥就是我们需要安装在远程主机上的。

>> 科普：~符号代表用户主目录，俗称家目录。其路径与当前登陆的用户有关，在 Linux 中普通用户家目录的路径是/home/用户名，而 root 用户是/root。Win­dows 10 中路径是C:\Users\用户名。在 ma­cOS 中路径是/Users/用户名。

#### 安装公钥

##### 从 GitHub 获取公钥

在 GitHub 密钥管理页面 添加公钥，比如我的用户名是 P3TERX，那么在主机上输入以下命令即可：

```bash
bash <(curl -fsSL git.io/key.sh) -g P3TERX
```

##### 从 URL 获取公钥

把公钥上传到网盘，通过网盘链接获取公钥：

```bash
bash <(curl -fsSL git.io/key.sh) -u https://p3terx.com/key.pub
```

##### 从本地文件获取公钥

通过 FTP 的方式把公钥传到 VPS 上，然后指定公钥路径：

```bash
bash <(curl -fsSL git.io/key.sh) -f ~/key.pub
```

#### 覆盖模式

使用覆盖模式（-o）将覆盖 /.ssh/authorized_keys 文件，之前的密钥会被完全替换掉，选项必须写在最前面才会生效，比如：

```bash
bash <(curl -fsSL git.io/key.sh) -o -g P3TERX
```

或者

```bash
bash <(curl -fsSL git.io/key.sh) -og P3TERX
```

#### 禁用密码登录

在确定使用密钥能正常登录后禁用密码登录：

```bash
bash <(curl -fsSL git.io/key.sh) -d
```

#### 修改 SSH 端口

把 SSH 端口修改为 2222：

```bash
bash <(curl -fsSL git.io/key.sh) -p 2222
```

#### 一键操作

安装密钥、修改端口、禁用密码登录一键操作：

```bash
bash <(curl -fsSL git.io/key.sh) -og P3TERX -p 2222 -d
```


---

> 作者: [枫](https://github.com/qiuzhi)  
> URL: https://blog.iqzhi.com/posts/linux-key-login/  

