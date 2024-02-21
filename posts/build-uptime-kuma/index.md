# 搭建uptime-kuma服务监控面板


随着在互联网VPS和家里软路由搭建了不少项目后，为了监控探活，及时通知报告，我物色到了一款叫`uptime-kuma`的服务监控面板。

<!--more-->

## 一 简介

uptime-kuma是一款开源监控工具，类似于“Uptime Robot”，UI简洁美观，支持TCP/PING/HTTP监控等，支持多语言其中包括中文。当服务出现故障时，可自动通过 Telegram、Discord、Gotify、Slack、Pushover、Email (SMTP) 等多种服务发送通知消息。

项目地址：https://github.com/louislam/uptime-kuma

## 二 Docker搭建

### 1 使用docker-compose

创建`docker-compose.yml`。

```yml
version: "3.0"
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
const whitelist = ["/bot1111111111:"];
const tg_host = "api.telegram.org";

addEventListener('fetch', event => {
    event.respondWith(handleRequest(event.request))
})

function validate(path) {
    for (var i = 0; i < whitelist.length; i++) {
        if (path.startsWith(whitelist[i]))
            return true;
    }
    return false;
}

async function handleRequest(request) {
    var u = new URL(request.url);
    u.host = tg_host;
    if (!validate(u.pathname))
        return new Response('Unauthorized', {
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
$ apt-get update && apt-get install vim -y             // 安装vim
$ vim /app/src/components/notifications/Telegram.vue   // 找到：api.telegram.org，将其替换成你的反代域名（就一处）
$ vim /app/server/notification-providers/telegram.js   // 找到：api.telegram.org，将其替换成你的反代域名（就一处）
$ exit 
$ docker restart uptime-kuma
```

## 三 参考资料

- [搭建uptime-kuma服务监控面板](https://nies.live/d/174)


---

> 作者: [枫](https://github.com/qiuzhi)  
> URL: https://blog.iqzhi.com/posts/build-uptime-kuma/  

