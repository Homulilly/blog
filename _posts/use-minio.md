---
title: 使用 Minio 管理图片等资源
date: 2017-11-22 16:05:12
tags:
 - Minio
categories:
 - Note
---

嗯，不同的网站下放的图片啥的，想放到一起管理，发现朋友用的 Minio 看起来不错，于是安装试了一下。  

使用 Nginx 配置好一个单独的子域名后，然后安装配置 Mino.

<!--more-->  
参考：[官方文档](https://docs.minio.io/docs/minio-quickstart-guide)

### 安装  
Ubuntu 可以直接下载执行文件  
```
sudo wget https://dl.minio.io/server/minio/release/linux-amd64/archive/minio.RELEASE.2021-06-17T00-10-46Z -O /opt/minio
sudo chmod +x /opt/minio
sudo ln -s /opt/minio /usr/bin/

minio --help
```

RELEASE.2021-06-17T00-10-46Z 最后内置 Minio Browser 的版本  
RELEASE.2022-05-26T05-48-41Z 最后支援 File System 的版本(使用 Minio Console，不支持域名 subpath)  


###  创建文件夹  
```
#存放文件
sudo mkdir /var/www/minio
```
~~使用下面的命令运行，使用输出的 AccessKey / SecretKey 登录，配置文件是 `/etc/minio/config.json`， AccessKey / SecretKey 会在里面保存，上传的文件会保存在 `/var/www/minio` 里。~~  

运行命令  

```
/usr/bin/minio server /var/www/minio
```

这时可以访问 `网站 IP:9000` 打开 Minio 的网页管理。 

默认的用户名密码是 minioadmin / minioadmin  

可以通过设置环境变量 `MINIO_ROOT_USER` and `MINIO_ROOT_PASSWORD` 指定

### 设置 Nginx    
使用域名进行访问，可以在 Nginx 的配置文件中加入下面的内容：  
```
# 反代 Minio Browser 
location / {
    proxy_set_header Host $http_host;
    proxy_pass http://localhost:9000;
}

# 禁止配置文件夹的访问
location ^~ /.minio.sys/ {
    deny all;
}

location ^~ /.minio.sys {
    deny all;
}


```

这时访问网站就会直接访问 Minio 的网页了。可以在 Minio 执行命令中加入 `--address "127.0.0.1:9000"`， 这样只会 Minio 只会监听本机地址。   

```
/usr/bin/minio server --address "127.0.0.1:9000" /var/www/minio
```

### 设置 Systemd 服务  
处理文件夹权限  
```
sudo chown -R www-data:www-data /var/www/minio
```

创建 `/etc/systemd/system/minio.service` 文件  

```
[Unit]
Description=minio_backend
After=network.target
[Service]
Environment="MINIO_ROOT_USER=[MINIO USER]"
Environment="MINIO_ROOT_PASSWORD=[MINIO PASSWORD]"
User=www-data
ExecStart=/usr/bin/minio server --address "127.0.0.1:9000" /var/www/minio
[Install]
WantedBy=multi-user.target  
```

载入并启动服务  
```
sudo systemctl daemon-reload
# 启动
sudo systemctl start minio
# 查看状态
sudo systemctl status minio
# 开机启动
sudo systemctl enable minio

# 查看输出
sudo journalctl -u minio
```

嗯...
这样，就是用 Minio 传文件，然后通过域名直接访问，比自己手动传方便了许多。感觉满足了自己的需求，而且还可以管理其他的文件，只是在 Minio 里图片点击就下载了，不能点开看的样子。

而且创建二层文件夹居然是通过直接 URL 访问然后上传文件的方式（
