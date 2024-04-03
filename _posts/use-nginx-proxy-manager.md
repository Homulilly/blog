---
title: 使用 Nginx Proxy Manager
date: 2024-03-02 12:29:58
categories:
 - Note
---

今天又看到有人在推荐 [Nginx Proxy Manger](https://nginxproxymanager.com/)(太长了，下文 NPM 指代)，于是我也在局域网的机器内装了一个，发现确实简单好用，完全的界面操作，非常便捷。  
而且支持使用 Cloudflare DNS 验证申请 Let's Encrypt 的通配符证书，也很方便设置访问限制。  

<!--more-->
### 安装

使用 Docker [安装](https://nginxproxymanager.com/setup/)是非常的简单，首先创建 `docker-compose.yml` 文件
```yml
version: '3.8'
services:
  app:
    image: 'jc21/nginx-proxy-manager:latest'
    restart: unless-stopped
    ports:
      # These ports are in format <host-port>:<container-port>
      - '80:80' # Public HTTP Port
      - '443:443' # Public HTTPS Port
      - '81:81' # Admin Web Port
      # Add any other Stream port you want to expose
      # - '21:21' # FTP

    volumes:
      - ./data:/data
      - ./letsencrypt:/etc/letsencrypt
```
然后 `docker compose up -d` 就可以访问 http://ip 测试了。

管理页面访问 http://ip:81 ，默认管理员账户如下，登入后即时修改。
```
Email:    admin@example.com
Password: changeme
```

### 申请证书

申请证书点击顶部的 `SSL Certificates` 再点击 `Add SSL Certificates` ，弹出窗口中依次填写
 - **Domain Names**: 要申请的证书域名 `*.domain.com` 
 - **Email Address for Let's Encrypt**: 申请 Let's Encrypt 使用的邮箱，
 - 打开 `Use a DNS Challenge` 选项
 - **DNS Provider**: 选择 Cloudflare 
 - **Credentials File Content**: 替换 API token，访问 [https://dash.cloudflare.com/profile/api-tokens](https://dash.cloudflare.com/profile/api-tokens) 创建，或是在域名概述首页，右下角 `API` 部分点击 `获取您的 API 令牌`
 - 最后勾选底部的同意 Let's Encrypt 的服务条款再点击 Save 就可以进行证书申请了

### 添加反代站点

在 `Hosts` > `Proxy Hosts` 中点击 `Add Proxy Host` 即可添加
![Add Proxy Host](https://m.nep.me/blog/post/use-npm-add-proxy-host.png)

`Domain Names` 填写要使用的域名，下面部分填写的是目标网站，需要注意的是如果使用的是 Docker 安装的 NPM ，反代本机的站点也不能使用 localhost 或是 127.0.0.1 ，要填写对应的局域网 IP，或是使用 `ip addr show docker0` 获取的 IP 地址 。
然后在 SSL 标签页中选择要使用的证书即可。  

### 保护管理页面
默认启动后管理网页的端口也是没有访问限制的，http://ip:81 可以直接访问，也没有证书，如果部署在公网中，这会很不安全，可以在改完配置后移除 `docker-compose.yml` 中对于 `Admin Web Port` 的转发，不过这样每次进行修改时都需要重启服务，略有麻烦。  

也可以使用 NPM 反代一下自己的管理端口，然后将 `docker-compose.yml` 配置中外部端口监听到公网无法访问的 IP 上，一开始我是设置为监听 `127.0.0.1:81` 发现访问报错，想了想意识到容器中的 NPM 是无法访问主机的 localhost 的，使用 `ip addr show docker0` 获得地址就可以了。 

然后再应用上 `Access Lists` 中的访问限制，就安全了不少。

### 问题
一设置 `Custom locations` ，站点就 offline 了，访问报错 `ERR_SSL_UNRECOGNIZED_NAME_ALERT`，搜了一下别人也有相同的问题  
[Github issues](https://github.com/NginxProxyManager/nginx-proxy-manager/issues/3536)

2024.03.02 说是补丁在路上了

