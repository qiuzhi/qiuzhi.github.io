# Docker Hub 连接指南


记录连接Docker Hub受阻后的吃用方法。

<!--more-->

## 一 简介

受网络影响，某地区可能连接Docker Hub受阻。

## 二 连接指南

### 1 配置镜像加速

创建或修改 /etc/docker/daemon.json 文件，修改为如下形式：

```json
{
    "registry-mirrors": [
        "https://docker.agsvpt.work",
        "https://docker.agsv.top",
        "http://hub-mirror.c.163.com",
        "https://docker.mirrors.ustc.edu.cn",
        "https://registry.docker-cn.com"
    ]
}
```

### 2 通过代理服务器

如OpenClash等。

1. 创建 dockerd 相关的 systemd 目录，这个目录下的配置将覆盖 dockerd 的默认配置

```bash
mkdir -p /etc/systemd/system/docker.service.d
```

2. 新建配置文件 /etc/systemd/system/docker.service.d/http-proxy.conf，这个文件中将包含环境变量

```bash
vi /etc/systemd/system/docker.service.d/http-proxy.conf

[Service]
Environment="HTTP_PROXY=http://proxy.example.com:80"
Environment="HTTPS_PROXY=https://proxy.example.com:443"
```

3. 如果你自己建了私有的镜像仓库，需要 dockerd 绕过代理服务器直连，那么配置 NO_PROXY 变量

```bash
[Service]
Environment="HTTP_PROXY=http://proxy.example.com:80"
Environment="HTTPS_PROXY=https://proxy.example.com:443"
Environment="NO_PROXY=your-registry.com,10.10.10.10,*.example.com"
```

## 三 重启Docker并确认设置成功

```bash
systemctl daemon-reload
systemctl restart docker.service

systemctl show --property=Environment docker
docker info
```

如此，这样配置后，应该可以正常拉取 docker 镜像。

## 四 自建镜像

### 使用 CloudFlare Worker 搭建

1. 在面板左侧找到 **Workers 和 Pages**，然后点击右侧的 **创建应用程序**、**创建 Worker**，修改一个好记的名字，**部署**

2. 接下来编辑代码，将 `worker.js` 的内容替换为下面内容

```js
import HTML from './docker.html';

export default {
    async fetch(request) {
        const url = new URL(request.url);
        const path = url.pathname;
        const originalHost = request.headers.get("host");
        const registryHost = "registry-1.docker.io";

        if (path.startsWith("/v2/")) {
        const headers = new Headers(request.headers);
        headers.set("host", registryHost);

        const registryUrl = `https://${registryHost}${path}`;
        const registryRequest = new Request(registryUrl, {
            method: request.method,
            headers: headers,
            body: request.body,
            // redirect: "manual",
            redirect: "follow",
        });

        const registryResponse = await fetch(registryRequest);

        console.log(registryResponse.status);

        const responseHeaders = new Headers(registryResponse.headers);
        responseHeaders.set("access-control-allow-origin", originalHost);
        responseHeaders.set("access-control-allow-headers", "Authorization");
        return new Response(registryResponse.body, {
            status: registryResponse.status,
            statusText: registryResponse.statusText,
            headers: responseHeaders,
        });
        } else {
        return new Response(HTML.replace(/{{host}}/g, originalHost), {
            status: 200,
            headers: {
            "content-type": "text/html"
            }
        });
        }
    }
}
```

这里相比原项目，将 **redirect: “manual”** 修改为了 **redirect: “follow”**，目的是为了让脚本自行处理 307 跳转，直接返回给我们跳转后的数据。
新建一个名为 `docker.html` 的 文件，内容如下

```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1" />
        <title>Mirror Usage</title>
        <style>
        html {
        height: 100%;
        }
        body {
        font-family: "Roboto", "Helvetica", "Arial", sans-serif;
        font-size: 16px;
        color: #333;
        margin: 0;
        padding: 0;
        height: 100%;
        display: flex;
        flex-direction: column;
        justify-content: space-between;

        }
        .container {
            margin: 0 auto;
            max-width: 600px;
        }

        .header {
            background-color: #438cf8;
            color: white;
            padding: 10px;
            display: flex;
            align-items: center;
        }

        h1 {
            font-size: 24px;
            margin: 0;
            padding: 0;
        }

        .content {
            padding: 32px;
        }

        .footer {
            background-color: #f2f2f2;
            padding: 10px;
            text-align: center;
            font-size: 14px;
        }
        </style>
    </head>
    <body>
        <div class="header">
        <h1>Mirror Usage</h1>
        </div>
        <div class="container">
        <div class="content">
            <p>镜像加速说明</p>
            <p>
            为了加速镜像拉取,你可以使用以下命令设置registery mirror:
            </p>
            <pre>
            sudo tee /etc/docker/daemon.json &lt;&lt;EOF
            {
                "registry-mirrors": ["https://{{host}}"]
            }
            EOF
            </pre>
            </br>
            <p>
            为了避免 Worker 用量耗尽,你可以手动 pull 镜像然后 re-tag 之后 push 至本地镜像仓库:
            </p>
            <pre>
            docker pull {{host}}/library/alpine:latest # 拉取 library 镜像
            docker pull {{host}}/coredns/coredns:latest # 拉取 library 镜像
            </pre>
        </div>
        </div>
        <div class="footer">
        <p>Powered by Cloudflare Workers</p>
        </div>
    </body>
</html>
```

3. 接下来，点击右上角的 **部署**，稍等片刻

4. 最后，返回面板，在 **设置**，**触发器** 处设置一个自己的域名，一切就大功告成了

不建议使用自带的 workers.dev 的域名，被墙了

## 五 注意

docker 镜像由 docker daemon 管理，所以不能用修改 shell 环境变量的方法使用代理服务，而是从 systemd 角度设置环境变量。


---

> 作者: [枫](https://github.com/qiuzhi)  
> URL: https://blog.iqzhi.com/posts/link-docker-hub/  

