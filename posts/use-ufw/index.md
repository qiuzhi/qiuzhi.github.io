# UFW 防火墙使用备忘


防火墙用于监控过滤传入和传出网络流量，可以定义一组规则用来放行或者组织特定流量。

很多大厂自带防火墙相当强力，但是对于大部分IDC商而言，并没有提供相应的面板，如果想要达到阻断某些流量的效果就需要自行安装并配置。比如22(ssh) 端口只允许特定IP登录。

Debian10 附赠UFW防火墙配置工具，用于管理iptables，不过更加友好，简单，易使用。

本文介绍如何使用UFW在Debian 系统中配置和管理防火墙。文章只包含基础用法。

本文环境：Debian11 默认root 权限

<!--more-->

## 使用UFW

### 安装UFW

```bash
apt update
apt install ufw
```

安装完成后可以使用 `ufw status verbose` 查看当前状态

### UFW默认策略

ufw防火墙默认阻止所有传入和转发流量，允许所有出站流量。这就意味着除非打开专门的ssh端口，否则所有访问服务器的流量，包括你自己都无法连接。

### 应用配置文件

```bash
# 查看软件安装包
ufw app list
# 查看相应配置文件
ufw app info 'SSH'
```

### 开启UFW

一定注意，使用UFW第一项就要开启你相应的ssh 端口，**否则下次将无法登录！**

```bash
# 按照默认配置开启tcp 22端口
ufw allow ssh
# 如果改变了ssh 端口，如33
ufw allow 33/tcp
# 开启ufw
ufw enable
```

## 通用规则

通过上文的开启ssh 端口并启用UFW，其实已经可以看出一般规则

### 增加规则

- 开启指定端口

```bash
# 端口以999为例
# 指定端口协议
ufw allow port/protoc
# 指定端口，不指定协议，将同时允许tcp，udp
ufw allow port
# 如开启999端口的udp 协议
ufw allow 999/udp
```

- 开启指定范围端口

```bash
# 开启80，440端口
ufw allow 80,443/tcp
# 开启888-999的tcp 协议端口
ufw allow 888:999/tcp
```

- 指定IP访问端口

```bash
# IP 66.66.66.66 可以访问连接所有服务器端口
ufw allow from 66.66.66.66
# 只有IP 66.66.66.66 可以访问22端口进行ssh连接
ufw allow from 66.66.66.66 to any port 22
```

- 子网/ip段 访问端口

```bash
# 192.168.1.1 - 192.168.1.254 可以访问3306 端口
ufw allow from 192.168.1.0/24 to any port 3306
```

- 拒绝访问

```bash
# 只需将allow 改为deny 即可
# 如拒绝1.2.3.4 访问网页
ufw deny from 1.2.3.4 to any port 80,443
```

- 通过服务开启端口

```bash
# 使用服务名称，如http
ufw allow http
# 指定协议端口
ufw allow 80/tcp
# 如安装了Nginx可使用
ufw allow 'Nginx HTTP'
```

### 查看当前已添加规则

使用 `ufw status numbered`

```bash
root@gogo:~# ufw status numbered
Status: active
 
     To                         Action      From
     --                         ------      ----
[ 1] 668/tcp                    ALLOW IN    Anywhere                  
[ 2] 80/tcp                     ALLOW IN    Anywhere                  
[ 3] 668/tcp (v6)               ALLOW IN    Anywhere (v6)             
[ 4] 80/tcp (v6)                ALLOW IN    Anywhere (v6)   
```

### 删除规则

- 使用编号删除

`ufw delete id`

```bash
root@gogo:~# ufw delete 2
Deleting:
 allow 80/tcp
Proceed with operation (y|n)? y
Rule deleted
```

- 使用端口删除

```bash
# 删除端口访问
ufw delete allow 80/tcp
```

### 设置 UFW 状态

```bash
# 禁用ufw
ufw disable
# 启用ufw
ufw enable
```
下一条命令为危险命令，将会**删除所有活动规则**，谨慎使用

`ufw reset`

```bash
root@gogo:~# ufw reset
Resetting all rules to installed defaults. This may disrupt existing ssh
connections. Proceed with operation (y|n)? 
```

## 问题解决

{{< admonition question "Docker 在UFW中的网络处理">}}

安装了Docker后，涉及Docker网络放通存在变化，可以查以下资料适配。

[解决-ufw-和-docker-的问题](https://github.com/chaifeng/ufw-docker#%E8%A7%A3%E5%86%B3-ufw-%E5%92%8C-docker-%E7%9A%84%E9%97%AE%E9%A2%98)

```bash
#命令备忘

#按container_name管理
ufw-docker allow nginx-proxy-manager
ufw-docker allow nginx-proxy-manager 443/tcp
ufw-docker delete allow nginx-proxy-manager

#放通docker容器内部
ufw allow from 172.16.0.0/12 to any
#放通zerotier
ufw allow from 10.147.x.0/24 to any
#放通本地
ufw allow from 192.168.x.0/24 to any
```

{{< /admonition >}}


---

> 作者:   
> URL: https://blog.iqzhi.com/posts/use-ufw/  

