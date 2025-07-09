---
title: 使用 File Browser 替代 AList (挂载本地存储)
date: 2025-06-13 13:27:29
tags:
 - File Browser
categories:
 - Note
---

最近 AList 被收购的话题热度很高，对于事件本身，我就不做评论了。不过对于收购方，基于他的历史情况，确实没有好感。  

与其使用历史版本，还是更换其他的项目比较好，最开始使用的 MinIO ，也是由于旧版本有漏洞才更换的 AList 。 

我的需求比较简单，**只是挂载本地的目录**，有个 Web 面板便于上传、管理一下图片等资源，并不需要挂载第三方网盘。  

搜索一下，发现 [File Browser](https://github.com/filebrowser/filebrowser) 这个项目比较符合我的需求，也支持 Docker 一键部署。 

不过这个项目也在寻找维护者：

> Warning!
> This project is currently not under active maintenance and is looking for maintainers. 

## File Browser 介绍

 - 支持 Docker 部署
 - 支持创建文件与文件夹
 - 支持显示文件列表
 - 支持用户管理
 - 支持预览图片
 - 支持编辑文本文件 
 - 支持创建分享链接
   - 支持设置分享期限
   - 支持设置密码
 - 使用 Go 语言编写，运行内存和 CPU 占用低

<!--more-->

## 使用 docker-compose 部署

如果安装好了 Docker 与 docker-compose ，部署起来上非常简单的。 

直接创建 docker-compose.yml 文件

```yml
services:
  filebrowser:
    image: filebrowser/filebrowser:s6
    container_name: filebrowser
    volumes:
      - /path/to/root:/srv
      - /path/to/filebrowser.db:/database/filebrowser.db
      - /path/to/settings.json:/config/settings.json
    environment:
      - PUID=$(id -u)
      - PGID=$(id -g)
    ports:
      - 8080:80
    restart: unless-stopped
```

启动前，如果是全新的环境，需要创建空白的 `/path/to/filebrowser.db` 文件，然后创建 `/path/to/settings.json` 填入下面的内容。  
```json
{
  "port": 80,
  "baseURL": "",
  "address": "",
  "log": "stdout",
  "database": "/database/filebrowser.db",
  "root": "/srv"
}
```
如果希望访问的路径不是 `/` 可以在 `baseURL` 中指定。

然后修改 docker-compose.yml 中 environment 的内容，在终端执行 `id -u` 获取，如需指定用户比如 `www-data` 则是 `id -u www-data` 
```yml
    environment:
      - PUID=$(id -u)
      - PGID=$(id -g)
```
该用户需要拥有 `/path/to/root` 的访问权限。  

执行 `docker-compose up -d` 启动服务。

访问 `https://<ip or domain>:8080` 登录，默认用户名密码都是 `admin` ，立刻在设置中修改，用户名也是可以修改的。  

设置 Nginx 反代，我直接使用了之前 AList 的内容，不需要修改：

```
location /filebrowser/ {
	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	proxy_set_header X-Forwarded-Proto $scheme;
	proxy_set_header Host $http_host;
	proxy_set_header X-Real-IP $remote_addr;
	proxy_set_header Range $http_range;     

	client_max_body_size 2048m;
	proxy_buffering off;
        proxy_pass http://127.0.0.1:8080;
    }
```

确认没有问题后，可以将 docker 容器的监听端口设置为 `127.0.0.1:8080`
```yml
    ports:
      - 127.0.0.1:8080:80
```

执行 `docker-compose down && docker-compose up -d` 重启容器