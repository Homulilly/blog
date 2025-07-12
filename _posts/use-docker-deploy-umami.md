---
title: 使用 Docker 部署 Umami 网站访问统计系统
date: 2025-07-10 11:16:35
tags:
 - Docker
 - Docker Compose
 - Selfhost
 - Umami
categories:
 - Note
---

## 关于 Umami 

[Umami](https://umami.is/) 是一个简单、快速、注重隐私且开源网站访问统计系统，可以作为 Google Analytics 的替代品，Umami 不使用 Cookie，不跟踪用户。   

Umami 支持自己部署，功能不受限制，如果自己没有服务器，也可以直接在官网注册使用。  

Umami 的界面非常简洁、直观，功能一目了然，这也是我使用 Umami 的原因，Google Analytics 太强大，界面也很复杂，有时找个功能要找半天。  

Umami 支持下面的功能
 - 统计浏览量、访问次数、访客数量、跳出率、平均访问时长
 - 按时间段生成柱状图
 - 查看每个 URL 对应的浏览量
 - 查看来源域名
 - 查看访客的浏览器、操作系统、设备类型、分辨率、地理位置
 - 支持生成各种报告

在开始使用之前，可以查看官方提供的 [DEMO](https://eu.umami.is/share/LGazGOecbDtaIwDr/umami.is) 体验后台与统计效果
![Umami Demo Preview](https://m.nep.me/blog/post/umami-demo-preview.png)

## 使用 Docker Compose 部署

使用 `docker compose` 可以直接一键部署，首先创建 `docker-compose.yaml`，填入下面的内容

<!--more-->
{% note info %}
没有 docker compose 可以查看 [Debian 12 / Ubuntu 24.04 安装 Docker 以及 Docker Compose 教程](https://u.sb/debian-install-docker/) 
{% endnote %}

使用的是官方提供的自带 postgresql 数据库版本的镜像

```yaml 
services:
  umami:
    image: ghcr.io/umami-software/umami:postgresql-latest
    ports:
      - "3000:3000"
    environment:
      # 使用自己的密码替换 <db_passwd> 
      DATABASE_URL: postgresql://umami:<db_passwd>@db:5432/umami
      DATABASE_TYPE: postgresql
      # 生成一段随机字符串替换 <app_secret>
      # openssl rand -base64 32
      APP_SECRET: <app_secret>
    depends_on:
      db:
        condition: service_healthy
    init: true
    restart: always
    healthcheck:
      test: ["CMD-SHELL", "curl http://localhost:3000/api/heartbeat"]
      interval: 5s
      timeout: 5s
      retries: 5
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: umami
      POSTGRES_USER: umami
      # 与前面生成的 <db_passwd> 一致
      POSTGRES_PASSWORD: <db_passwd>
    volumes:
      # 数据文件保存在 docker-compose.yaml 所在目录下的 data 文件夹
      - ./data:/var/lib/postgresql/data
    restart: always
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U $${POSTGRES_USER} -d $${POSTGRES_DB}"]
      interval: 5s
      timeout: 5s
      retries: 5
```

然后运行下面的命令即可自动拉取镜像启动
```sh 
docker compose up -d
```

启动后，访问 `http://<你的服务器 IP>:3000` 进入后台，默认用户名是 `admin` 密码 `umami` ，可以在右上角 切换界面语言。

{% note danger %}
登入后 请第一时间前往 设置 > 用户 修改用户名与密码
{% endnote %}

## 设置 Nginx 反代
Nginx 配置文件中的反代部分如下
```
location / {
    proxy_pass http://127.0.0.1:3000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    proxy_http_version 1.1;
    proxy_read_timeout 60s;
}
```

然后为了安全，我们可以将 docker-compose.yaml 文件中的端口设置仅限本地 Nginx 访问，将下面部分
```
    ports:
      - "3000:3000"
```
修改为
```
    ports:
      - "127.0.0.1:3000:3000"
```

## 添加站点

点击 **设置** > **添加网站** 进行添加：
 - **名字**：显示在 Umami 后台，用于辨识
 - **域名**：填入实际访问域名，例如 `example.com`，不需要加 `https://`

然后点击对应网站的 **编辑**，在 **跟踪代码** 部分即可查看需要使用的代码，将对应的代码附加到需要追踪的网页中即可。  

```js
<script defer src="https://example.com/script.js" data-website-id="xxxx-xxxx-xxxx-xxxx"></script>
```

我使用的 Hexo Theme NEXT 主题，可以直接在主题设置中填写 **域名** 与 `data-website-id` 让追踪生效。

## 资源占用

使用 docker stats 查看资源占用情况如下

```
CONTAINER ID   NAME               CPU %     MEM USAGE / LIMIT     MEM %     NET I/O           BLOCK I/O         PIDS 
df307a49da94   umami-umami-1      0.01%     238.8MiB / 1.933GiB   12.06%    16.8MB / 655kB    48MB / 73.8MB     48 
e7c0ca63f761   umami-db-1         0.02%     38.71MiB / 1.933GiB   1.96%     189kB / 162kB     8.56MB / 1.26MB   10 
```