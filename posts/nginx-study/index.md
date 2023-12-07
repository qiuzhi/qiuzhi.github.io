# Nginx学习笔记


Nginx的用法、配置及问题解决。

<!--more-->

## nginx 配置 http 请求重定向到 https

```conf
server {
    listen 80;
    server_name aaa.abc.dd;
    rewrite ^(.*)$ https://${server_name}$1 permanent;
}
```

这样就可以 http://aaa.abc.dd 转到 https://aaa.abc.dd 了。

## Nginx防止被域名恶意解析的配置

nginx在决定请求由哪个server块执行时，主要关注的是server块中的listen和server_name两个字段，如果根据listen指令无法得到最佳匹配，将会开始解析server_name指令。nginx会检查请求中的"Host"头，这个值包含了客户端实际试图请求的域名或者ip地址。nginx会根据这个值去匹配server_name指令，匹配规则会在文章中详细描述。其中有一个需要大家注意的地方是如果没有匹配到任何规则的话，则会选择可用列表中的第一个server，带来的问题就是未绑定域名或IP直接访问80和443端口会给后端逻辑服务增加压力并产生不合理的错误日志，合适的解决办法是通过在nginx的server块中添加default_server禁止未绑定域名或IP访问80和443端口过滤不合理的流量。

```conf
server {
    server_name _;
    listen 80 default_server;
    listen 443 ssl default_server;

    ## To also support IPv6, uncomment this block
    # listen [::]:80 default_server;
    # listen [::]:443 ssl default_server;

    ssl_certificate <path to cert>;
    ssl_certificate_key <path to key>;

    return 444;

    access_log /var/log/nginx/000_default.access.log;
    error_log /var/log/nginx/000_default.error.log;
}
```

这样在浏览器端访问的时候，浏览器会自动提示用户无法访问。

## Server_name指令

如果根据listen指令无法得到最佳匹配,将会开始解析server_name指令.nginx会检查请求中的"Host"头,这个值包含了客户端实际试图请求的域名或者ip地址.nginx会根据这个值去匹配server_name指令,匹配规则如下:

nginx会尝试寻找一个和sever_name和Host值完全匹配的server块,如果找到多个精确匹配,则会使用第一个匹配的server块
如果没有找到精确匹配的server块,则nginx尝试找到server_name带有*开头的server块,如果找到多个,则选择最长匹配的server块
如果没有找到使用开头的server块,则会寻找以结尾的server块,同样,如果有多个匹配, 选择最长匹配
如果没有找到使用*匹配的server块,则会寻找使用正则表达式(以~开头)定义server_name的server块,如果找到多个匹配,会使用第一个匹配
如果没有找到正则表达式匹配的server块,则nginx将会选择一个匹配listen字段的default server块.每一个ip和端口组合都可以配置一个且只能配置一个默认的default_server块,如果没有的话,则会选择可用列表中的第一个server


## 常用命令

```bash
# reload nginx
nginx -t
nginx -s reload
```


---

> 作者:   
> URL: https://blog.iqzhi.com/posts/nginx-study/  

