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
- Gitlab-CE 16.3

16.3 的版本简体中文翻译进度 98%

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
mkdir -p /opt/gitlab/vol
```

2. 创建环境变量文件
```
cd /opt/gitlab 

vim .env
```
填入下面的内容
```
GITLAB_HOME=/opt/gitlab/vol
```

## 使用 Docker Compose 安装 Gitlab

1. 进入工作目录
```
cd /opt/gitlab
```

2. 创建 `docker-compose.yml` 配置文件
使用社区版镜像 `gitlab/gitlab-ce` ，是免费的
```bash
version: '3.6'
services:
  web:
    image: 'gitlab/gitlab-ce:latest'
    container_name: 'gitlab'
    restart: always
    hostname: '10.5.5.5'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://10.5.5.5'
        gitlab_rails["time_zone"] = 'Asia/Shanghai'
        gitlab_rails['gitlab_ssh_host'] = '10.5.5.5'
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
第一次登陆使用用户 `root` ，密码在 `/opt/gitlab/vol/config/initial_root_password` 中查看，登入后可以在后台修改密码  
登入后可以按照顶部提示关闭允许注册选项    
`Settings` > `General` > `Sign-up restrictions`

## 首页预览 
![GITLAB-CE HOME](https://m.nep.me/blog/post/gitlab-home.png)

## 参考  
- [Centos 使用Docker-compose搭建私有Gitlab](https://cloud.tencent.com/developer/article/1924734)
- [How to Install GitLab CE with Docker on Debian 12](https://www.howtoforge.com/how-to-install-gitlab-with-docker-on-debian-12/)