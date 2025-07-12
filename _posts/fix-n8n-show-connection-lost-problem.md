---
title: 解决 n8n 显示 Connection lost 的错误
date: 2025-07-11 19:02:55
tags:
 - n8n
categories:
 - Note
---

我刚装好的 n8n 实例，版本 1.101.1 ，后台访问没有问题，但是创建 workflow 的时候右上角提示 Connection lost
```
You have a connection issue or the server is down.
n8n should reconnect automatically once the issue is resolved.
```

![n8n show connection lost](https://m.nep.me/blog/post/n8n-show-error.png)  

先说一下我的环境，我是使用 docker 安装的 n8n，然后使用了 Nginx 进行的反代，不是使用官方提供的带 Traefik 的 docker compose 文件。 

搜索了一下，发现是 n8n 后端的检测方式的问题，可以修改为使用 websocket 检测的方式：

<!--more-->

在 docker-compose.yaml 中加入 `N8N_PUSH_BACKEND=websocket`

```yaml
services:
  n8n:
    image: docker.n8n.io/n8nio/n8n
    container_name: n8n
    restart: always
    ports:
      - "127.0.0.1:5678:5678"
    environment:
      # ...
      # 添加下面这一行
      - N8N_PUSH_BACKEND=websocket
      # ...
```

然后在 Nginx 中的反代部分启用 websocket 的支持

```
    location / {
        proxy_pass http://127.0.0.1:5678;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;

        # 添加下面三行
        proxy_http_version 1.1; 
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
```

重启 n8n 

```sh
docker compose down 
docker compose up -d
```
以及重载 nginx
```
nginx -t
nginx -s reload
```

然后就可以了 

![n8n error fixed](https://m.nep.me/blog/post/n8n-error-fixed.png)  