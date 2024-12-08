# 哪吒监控V1升级注意事项


[哪吒监控](https://nezha.wiki)，开发大佬给力，发版频繁，也就伴随着可能的不稳定，生存在风雨飘摇中，也随着这次升级V1较稳定的版本，记录下升级的注意点及常用配置备份，供参考。

&lt;!--more--&gt;

## 一 注意点（对比V0与V1）

1. 不要想着从 V0 无痛迁移，想什么呢？人家都说了 **V0 和 V1 的数据库结构不兼容**，哪来的跨版本升级。
2. V1 安装完默认本地账密登录，所以请**第一时间更改管理员账号密码**。
3. V1 版本不再区分 Dashboard 和 gRPC 端口，访问与通信均通过默认的 **8008** 端口。
4. 若 V1 使用了 CDN，请确保 **CDN 服务商需支持 WebSocket 服务**，并已正确启用 WebSocket 功能。
5. **V1 自带简单的 WAF**，可通过路径 /dashboard/settings/waf 进行管理配置，干啥用呢？防止外界暴力破解登录接口。

## 二 使用 Nginx Proxy Manager 反代配置

正常填写 Scheme，IP，Port。

选项点开 Websockets Support。

Advanced 选项卡加入以下配置：

```conf
underscores_in_headers on;

# websocket 相关
location ~* ^/api/v1/ws/(server|terminal|file)(.*)$ {
    proxy_set_header Host $host;
    proxy_set_header nz-realip $remote_addr;
    proxy_set_header Origin https://$host;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection &#34;upgrade&#34;;
    proxy_read_timeout 3600s;
    proxy_send_timeout 3600s;
    proxy_pass http://$server:$port;
}
# grpc 相关    
location ^~ /proto.NezhaService/ {
    grpc_set_header Host $host;
    grpc_set_header nz-realip $remote_addr;
    grpc_read_timeout 600s;
    grpc_send_timeout 600s;
    grpc_socket_keepalive on;
    client_max_body_size 10m;
    grpc_buffer_size 4m;
    grpc_pass grpc://$server:$port;
}
# web
location / {
    proxy_set_header Host $host;
    # proxy_set_header nz-realip $http_cf_connecting_ip; # 替换为你的 CDN 提供的私有 header，此处为 CloudFlare 默认
    proxy_set_header nz-realip $remote_addr; # 如果你使用nginx作为最外层，就把上面一行注释掉，启用此行
    proxy_read_timeout 3600s;
    proxy_send_timeout 3600s;
    proxy_buffer_size 128k;
    proxy_buffers 4 256k;
    proxy_busy_buffers_size 256k;
    proxy_max_temp_file_size 0;
    proxy_pass http://$server:$port;
}
```

## 三 各省节点备份

- [全国 34 省份 TCPING v4 IP](https://wph.im/259.html)
- [【存】全国各省份三网 TCP-Ping IPv4 地址](https://www.nodeseek.com/post-68572-1)

## 四 告警规则生成工具

- [Nezha Rules Generator](https://nz.sina.us.kg/)
- [哪吒监控流量警告规则生成器](https://wiziscool.github.io/Nezha-Traffic-Alarm-Generator/)

---

> 作者: [枫](https://github.com/qiuzhi)  
> URL: https://blog.iqzhi.com/posts/nezha-v1/  

