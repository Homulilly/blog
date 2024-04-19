---
title: 自建 Bitwarden 密码管理服务
date: 2020-06-06 16:05:18
tags:
 - Bitwarden
categories: 
 - Note
---

以前一直用的 Keepass + Dropbox 同步，手机上使用起来还是不太麻烦。

前一阵子看见别人分享了 [Bitwarden](https://bitwarden.com/) 这个密码管理软件，服务端都是开源的，所以可以自建。

自己搭了一下，用起来还可以。

- **不用花钱**，自建的服务端高级功能都是免费的
- 全平台的客户端、浏览器扩展支持
- 密码管理中支持 TOTP
- 使用 Docker 搭建起来还挺简单的
- Keepass 的数据可以通过导出 xml 文件导入 Bitwarden，不过不能导入附件

这里备份一下安装过程
<!--more-->
## 安装 docker

```
# 1. 执行官方的安装脚本
wget -qO- get.docker.com | bash

# 2. 检查版本
docker version

# 3. 启动 Docker
systemctl start docker

# 4. 查看 Docker 状态
systemctl status docker

# 5. 设置 Docker 自启动
systemctl enable docker
```

## 安装 Bitwarden

### 1.获取 bitwarden_rs 镜像
```
docker pull bitwardenrs/server:latest
```

Update 2023.01.09:  
解决自建 Bitwarden 服务端在 Chrome 扩展更新后无法登陆的问题  

```
docker pull vaultwarden/server:latest
```

> 参考 Chrome 商店评论: 
> "the bitwardenrs/server docker image is DEPRECATED, use vaultwarden/server image with the 1.27.0 version, it's working."  by Xiao Yu

后续运行命令也要修改为对应的镜像  


### 2.运行服务端

Tips: 建议使用 `docker compose` 运行与管理

```
docker run -d --name bitwarden -v [Bitwarden 的数据存储路径]:/data/ -p 6666:80 vaultwarden/server:latest	
```
默认会开启用户注册，毕竟自己使用时也要注册一个用户，自己注册完了之后可以在命令中加入下面的内容关闭注册

```
-e SIGNUPS_ALLOWED=false
```

### 3.设置 Nginx 反代

```
location / {
    #...

    proxy_pass http://localhost:6666;
    
    #...
}
```

### 4.启动、停止服务端
```
docker start/stop bitwarden
```

## Docker 命令
```
# $name 为 docker run 中定义的 name
# 启动容器
docker start $name

# 停止容器
docker stop $name

# 删除容器
docker rm $name

# 查看运行容器
docker ps -as
```

## 更新
```
# 1. 重新获取镜像
#docker pull bitwardenrs/server:latest
docker pull vaultwarden/server:latest

# 2. 停止、删除原容器
docker stop bitwarden
docker rm bitwarden

# 3. 重新运行 docker run 命令

# 4. 查看镜像文件
docker image ls

# 5. 删除原镜像文件, $ID 在 step4 中可以看到
docker image rm $ID
```

设置 docker 容器开机自启：
 
```
# 设置 docker 开机启动
systemctl enable docker 

# 设置容器自动重启
docker run -d --restart always --name 容器名称 镜像  
```
  - no //默认，容器退出后不重启
  - on-failure //非正常退出后重启
  - on-failure:3 // 非正常退出时重启，最多三次
  - always //无论退出状态如何，都会重启
  - unless-stopped //在容器退出时总是重启容器，但是不考虑在 Docker 守护进程启动时就已经停止了的容器 

更新已有容器：  

```
docker update --restart=always 容器 ID / 容器名称
```

参考: [搭建自己的密码管理服务器 Bitwarden](https://cloud.tencent.com/developer/article/1578102)

## 使用 `docker compose`
每次更新都需要删除再重建，确实不便，尤其是使用了多个 docker 服务的时候，可以使用 `docker compose` 管理。
现在 `docker compose` 已内置进 docker cli 命令中，无需手动安装 

创建 `docker-compose.yaml` 文件，填入下面的内容

```yaml docker-compose.yaml
version: '3'
services:
  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    ports: 
      - 127.0.0.1:6666:80
    environment:
      # WEBSOCKET_ENABLED: "true"  # Enable WebSocket notifications.
      SIGNUPS_ALLOWED: "false"
      INVITATIONS_ALLOWED: "false"
    volumes: 
      - [Bitwarden 的数据存储路径]:/data/
    restart: always
```

随后运行启动命令就可以了  
- `docker-compose build`: 启动构建，构建在 yml 文件中定义的所有镜像
- `docker compose up`: 启动容器
- `docker compose up -d`: 启动容器并后台运行
- `docker compose down`: 停止由 up 命令启动的所有容器
- `docker compose ps`: 列出由 up 命令启动的所有容器
- `docker compose images`: 由 build 命令构建的所有镜像
- `docker compose logs`: 查询日志
- `docker compose start <service>`: 启动 yml 中定义的单个服务，使用服务名，不是容器名
- `docker compose stop <service>`: 停止 yml 中定义的单个服务
- `docker compose rm <service>`: 删除已停止的某个服务
- `docker compose config`: 验证 yml 文件有效性

`docker-compose.yaml` 文件中可以同时填写多个容器，更新单个容器可以执行下面的操作
```sh
# 更新所有镜像
docker compose pull  
docker compose up -d 

# 更新单个镜像
docker compose pull $name 
docker compose stop $name
docker compose rm $name
docker compose up -d

# 删除历史镜像
docker image prune
```

如果运行 `docker compose pull` 时报下面错误
```sh
error getting credentials - err: exit status 1, out: `GDBus.Error:org.freedesktop.DBus.Error.
ServiceUnknown: The name org.freedesktop.secrets was not provided by any .service files`
```
安装下面软件包即可
```sh
apt install gnupg2 pass
```

参考: [使用Docker-Compose管理多容器应用](https://www.cnblogs.com/xhy0826/p/Docker-Compose-ASPNETCORE.html)