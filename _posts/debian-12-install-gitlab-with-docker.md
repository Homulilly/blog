---
title: 在 Debian 12 中使用 Docker 安装 Gitlab
date: 2023-08-23 20:42:01
categories:
 - Note
tags:
 - Debian
 - Gitlab
 - Docker
---

有些内容不需要推送到远端服务器，正好之前将一台零刻小主机作为服务器使用，于是自己搭一个 Gitlab 的服务器，使用社区版是免费的。

Linux 虚拟机的内存要设置的大一些，Gitlab 会使用 3GB+ 的内存。

- Debian 12.01
- Gitlab-CE 16.5

16.3 的版本简体中文翻译进度 99%

<!--more-->

## 准备  
首先更新系统与软件，然后安装所需的依赖
```
apt update && apt upgrade

apt install ca-certificates curl openssh-server apt-transport-https gnupg lsb-release -y
```

## 安装 Docker 与 Docker Compose

```
apt install docker docker-compose -y
```

Docker 的 Gitlab 也需要使用 SSH 端口，可以将系统的 SSH 端口修改其他数字

```
vim /etc/ssh/sshd_config
```

取消注释并修改端口数字
```
Port 22
```

重启 SSH 服务生效
```
systemctl restart sshd
```


## 设置工作目录
1. 创建目录  
```
mkdir -p /opt/gitlab
```

2. 创建环境变量文件
```
cd /opt/gitlab 

vim .env
```
填入下面的内容
```
GITLAB_HOME=/opt/gitlab
```

## 使用 Docker Compose 安装 Gitlab

1. 进入工作目录
```
cd /opt/gitlab
```

2. 创建 `docker-compose.yml` 配置文件
使用社区版镜像 `gitlab/gitlab-ce` ，是免费的，配置中的 `gitlab.example.com` 替换为使用的域名，或是服务器的 IP 
```bash
version: '3.6'
services:
  web:
    image: 'gitlab/gitlab-ce:latest'
    container_name: 'gitlab'
    restart: always
    hostname: 'gitlab.example.com' # or IP
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'gitlab.example.com'
        gitlab_rails["time_zone"] = 'Asia/Shanghai'
        gitlab_rails['gitlab_ssh_host'] = 'gitlab.example.com'
        gitlab_rails['gitlab_shell_ssh_port'] = 22
        # Add any other gitlab.rb configuration here, each on its own line
    ports:
      - '80:80'
      - '443:443'
      - '22:22'
    volumes:
      - '$GITLAB_HOME/config:/etc/gitlab'
      - '$GITLAB_HOME/logs:/var/log/gitlab'
      - '$GITLAB_HOME/data:/var/opt/gitlab'
    shm_size: '256m'
```
物理目录 > 对应 Docker 镜像内部目录
- $GITLAB_HOME/data > /etc/gitlab 
- $GITLAB_HOME/logs	> /var/log/gitlab
- $GITLAB_HOME/config > /var/opt/gitlab

3. 启动镜像
```
docker-compose up
```
后台启动
```
docker-compose up -d
```
启动需要等待一些时间   
第一次登陆使用用户 `root` ，密码在 `/opt/gitlab/config/initial_root_password` 中查看，登入后可以在后台修改密码  
登入后可以按照顶部提示关闭允许注册选项    
`Settings` > `General` > `Sign-up restrictions`

## 首页预览 
![GITLAB-CE HOME](https://m.nep.me/blog/post/gitlab-home.png)

## 设置 SSL 证书

如果是普通服务器上部署是支持的 `Let's Encrypt` 的 : [Integrating a free certificate from Let's Encrypt](https://github.com/danieleagle/gitlab-https-docker#integrating-a-free-certificate-from-lets-encrypt) 

我是在局域网内使用，购买了一份证书后手动获取了证书文件 `server-cert.pem` 与 `server-key.key` ，在 `$GITLAB_HOME` 中创建 ssl 文件夹，将证书文件拷贝到其中，然后编辑 `docker-compose.yml` 文件，添加相关配置。

最终配置文件如下，将没有修改的行添加了注释，可以直观的查看新增/修改内容。

```bash
#version: '3.6'
#services:
#  web:
#    image: 'gitlab/gitlab-ce:latest'
#    container_name: 'gitlab'
#    restart: always
#    hostname: 'gitlab.example.com'
#    environment:
#      GITLAB_OMNIBUS_CONFIG: |
        external_url 'https://gitlab.example.com'
#        gitlab_rails["time_zone"] = 'Asia/Shanghai'
#        gitlab_rails['gitlab_ssh_host'] = 'gitlab.example.com'
#        gitlab_rails['gitlab_shell_ssh_port'] = 22
#        # Add any other gitlab.rb configuration here, each on its own line
        nginx['listen_port'] = 443
        nginx['redirect_http_to_https'] = true
        nginx['ssl_certificate'] = "/etc/ssl/certs/gitlab/gitlab.example.com/server-cert.pem"
        nginx['ssl_certificate_key'] = "/etc/ssl/certs/gitlab/gitlab.example.com/server-key.key"
        nginx['ssl_ciphers'] = "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384"
        nginx['ssl_protocols'] = "TLSv1.2 TLSv1.3"
#    ports:
#      - '80:80'
#      - '443:443'
#      - '22:22'
#    volumes:
#      - '$GITLAB_HOME/config:/etc/gitlab'
#      - '$GITLAB_HOME/logs:/var/log/gitlab'
#      - '$GITLAB_HOME/data:/var/opt/gitlab'
      - '$GITLAB_HOME/ssl:/etc/ssl/certs/gitlab'
#    shm_size: '256m'
```

重启服务，访问 `https://` 查看是否生效。
```bash
# 关闭服务
docker-compose down

# 后台启动
docker-compose up -d

# 查看日志
docker-compose logs -f
```

GitLab 启动提示
```
gitlab | Current version: gitlab-ce=16.9.1-ce.0
gitlab | 
gitlab | Configure GitLab for your system by editing /etc/gitlab/gitlab.rb file
gitlab | And restart this container to reload settings.
gitlab | To do it use docker exec:
gitlab | 
gitlab |   docker exec -it gitlab editor /etc/gitlab/gitlab.rb
gitlab |   docker restart gitlab
gitlab | 
gitlab | For a comprehensive list of configuration options please see the Omnibus GitLab readme
gitlab | https://gitlab.com/gitlab-org/omnibus-gitlab/blob/master/README.md
gitlab | 
gitlab | If this container fails to start due to permission problems try to fix it by executing:
gitlab | 
gitlab |   docker exec -it gitlab update-permissions
gitlab |   docker restart gitlab
```

## 升级后无法启动

Gitlab 部分版本不可以 [跨版本升级](https://docs.gitlab.com/ee/update/#upgrade-paths)，我是从 16.5.x 升级到 16.9.x 提示我需要先升级至 16.7.x 再更新至 16.9.x

尝试了一下确实可以，可以在 [Docker Hub](https://hub.docker.com/r/gitlab/gitlab-ce/tags) 查看历史版本号 

## 参考  
- [Centos 使用 Docker-compose 搭建私有Gitlab](https://cloud.tencent.com/developer/article/1924734)
- [How to Install GitLab CE with Docker on Debian 12](https://www.howtoforge.com/how-to-install-gitlab-with-docker-on-debian-12/)
- [GitLab with HTTPS on Docker](https://github.com/danieleagle/gitlab-https-docker/blob/master/docker-compose.yml)