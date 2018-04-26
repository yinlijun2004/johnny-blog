---
title: 通过IP地址访问nginx服务
date: 2018-04-25 15:29:24
tags: [nginx, nodejs]
---

使用nginx设置反向代理的时候，如果没有域名，可以通过如下方式设置。

#### 删除默认站点

将/etc/nginx/sites-available/default的内容注释掉。

将/etc/nginx/sites-enabled/default的内容注释掉。

### 添加站点

在/etc/nginx/conf.d/下创建一个配置，比如ota.conf，内容如下：
<!-- more -->

```json
upstream ota_server {
  server 127.0.0.1:3020;
}

server {
  listen 80 default_server;
  listen [::]:80 default_server;
  access_log /var/log/nginx/ota_server.log;

  location / {
    proxy_pass http://127.0.0.1:3020;
    proxy_http_version 1.1;
    root /home/otaserver/project/ota_sever/public;
    index index.html;
    proxy_set_header Host $http_host;
    proxy_set_header X-Forwarded-For $remote_addr;
  }
}
```

这样，就能通过ip访问到ota_server这个服务了。