# 搭建uptime-Kuma服务监控面板


随着在互联网VPS和家里软路由搭建了不少项目后，为了监控探活，及时通知报告，我物色到了一款叫`uptime-kuma`的服务监控面板。

&lt;!--more--&gt;

## 一 简介

uptime-kuma是一款开源监控工具，类似于“Uptime Robot”，UI简洁美观，支持TCP/PING/HTTP监控等，支持多语言其中包括中文。当服务出现故障时，可自动通过 Telegram、Discord、Gotify、Slack、Pushover、Email (SMTP) 等多种服务发送通知消息。

项目地址：https://github.com/louislam/uptime-kuma

## 二 Docker搭建

### 1 使用docker-compose

创建`docker-compose.yml`。

```yml
version: &#34;3.0&#34;
services:
  uptime-kuma:
    image: louislam/uptime-kuma:latest
    container_name: uptime-kuma
    volumes:
      - ./data/:/app/data
    ports:
      - 3037:3001
    restart: always
```

然后`docker-compose up -d`。

### 2 设置Telegram消息通知

#### 2.1 设置TG消息通知

点击新增，创建监控项，填写 Bot Token 和 Chat ID 即可配置好 Telegram 消息通知。

#### 2.2 解决国内部署无法发送TG通知的问题

##### 2.2.1 CloudFlare workers创建workers

```csharp
const whitelist = [&#34;/bot1111111111:&#34;];
const tg_host = &#34;api.telegram.org&#34;;

addEventListener(&#39;fetch&#39;, event =&gt; {
    event.respondWith(handleRequest(event.request))
})

function validate(path) {
    for (var i = 0; i &lt; whitelist.length; i&#43;&#43;) {
        if (path.startsWith(whitelist[i]))
            return true;
    }
    return false;
}

async function handleRequest(request) {
    var u = new URL(request.url);
    u.host = tg_host;
    if (!validate(u.pathname))
        return new Response(&#39;Unauthorized&#39;, {
            status: 403
        });
    var req = new Request(u, {
        method: request.method,
        headers: request.headers,
        body: request.body
    });
    const result = await fetch(req);
    return result;
}
```

##### 2.2.2 修改uptime-kuma的代码

```bash
$ docker exec -it uptime-kuma /bin/bash
$ apt-get update &amp;&amp; apt-get install vim -y             // 安装vim
$ vim /app/src/components/notifications/Telegram.vue   // 找到：api.telegram.org，将其替换成你的反代域名（就一处）
$ vim /app/server/notification-providers/telegram.js   // 找到：api.telegram.org，将其替换成你的反代域名（就一处）
$ exit 
$ docker restart uptime-kuma
```

##### 2.2.3 进入docker内部运行apt-get update报错

原因是由于Debian官方将Debian10(Debian buster)软件源由默认站点deb.debian.org移至存档站点archive.debian.org，基于官方软件源的镜像站同步了这一修改， 导致20250723之前的Debian10镜像在运行apt update命令时会报错： buster Release 404 Not Found 。 

解决方法是：切换至Debian归档仓库（维持Buster）

适用场景：需保留Debian 10环境，不升级系统。

修改镜像源：

```bash
mv /etc/apt/sources.list /etc/apt/sources.list-bak
cat &gt;/etc/apt/sources.list
```

1. 官方归档地址：

```bash
deb https://archive.debian.org/debian buster main contrib non-free
deb-src https://archive.debian.org/debian buster main contrib non-free
deb https://archive.debian.org/debian-security buster/updates main contrib non-free
deb-src https://archive.debian.org/debian-security buster/updates main contrib non-free
deb https://archive.debian.org/debian buster-updates main contrib non-free
deb-src https://archive.debian.org/debian buster-updates main contrib non-free
```

2. 阿里云镜像软件源：

```bash
deb https://mirrors.aliyun.com/debian-archive/debian buster main contrib non-free
deb-src https://mirrors.aliyun.com/debian-archive/debian buster main contrib non-free
deb https://mirrors.aliyun.com/debian-archive/debian-security buster/updates main contrib non-free
deb-src https://mirrors.aliyun.com/debian-archive/debian-security buster/updates main contrib non-free
deb https://mirrors.aliyun.com/debian-archive/debian buster-updates main contrib non-free
deb-src https://mirrors.aliyun.com/debian-archive/debian buster-updates main contrib non-free
```

## 三 参考资料

- [搭建uptime-kuma服务监控面板](https://nies.live/d/174)
- [Debian 10 执行 sudo apt update 报错的解决办法](https://www.cnblogs.com/rnckty/p/19021679)


---

> 作者: [枫](https://github.com/qiuzhi)  
> URL: https://blog.iqzhi.com/posts/build-uptime-kuma/  

