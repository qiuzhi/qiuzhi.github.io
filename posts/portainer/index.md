# Portainer - Docker可视化管理工具


配置Docker可视化管理工具。

<!--more-->

## 连接远程和管理docker

远程管理docke，端口默认是2375，是未加密的docker socket，远程root无密码访问主机

### 开启配置

#### 方法一

配置远程访问的API

```bash
vim /etc/default/docker

增加一行：
DOCKER_OPTS="-H tcp://0.0.0.0:2375"
```

> PS：这是网上给的配置方法，也是这种简单配置让Docker Daemon把服务暴露在tcp的2375端口上，这样就可以在网络上操作Docker了。Docker本身没有身份认证的功能，只要网络上能访问到服务端口，就可以操作Docker。

#### 方法二

在/usr/lib/systemd/system/docker.service，配置远程访问。

主要是在`[Service]`这个部分，找到 ExecStart字段修改如下

```bash
vim /usr/lib/systemd/system/docker.service

#ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H fd:// --containerd=/run/containerd/containerd.sock
```

#### 方法三

修改`daemon.json`的配置

```bash
vim /etc/docker/daemon.json

{
  "hosts": ["tcp://0.0.0.0:2375", "unix:///var/run/docker.sock"]
}
```

### 重启docker，配置防火墙

1. 重启docker重新读取配置文件，重新启动docker服务

```bash
systemctl daemon-reload
systemctl restart docker
```

2. 开放防火墙端口

```bash
firewall-cmd --zone=public --add-port=2375/tcp --permanent
#或
ufw allow from 192.168.9.0/24 to any
```

3. 刷新防火墙

```bash
firewall-cmd --reload
#或
systemctl restart ufw
```

4. 如果重启起不来

估计是这个 `unix://var/run/docker.sock` 文件位置不对，查找一下正确位置就好了。

```bash
find / -name docker.sock
```


---

> 作者: [枫](https://github.com/qiuzhi)  
> URL: https://blog.iqzhi.com/posts/portainer/  

