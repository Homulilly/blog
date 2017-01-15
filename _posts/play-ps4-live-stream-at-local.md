---
title: 搭建 PS4 本地直播服务器
date: 2016-12-21 01:35:23
tags:
 - Nginx
 - PS4
categories:
 - Note
---

总会有那么点奇怪的需求，本来是因为如龙 6 不能截图，想折腾一下，然而开始折腾之前就又想到了既然不能截图，直播肯定也不能，坑爹的世嘉，不能截妹子，我 Share 键要着有啥用。但还是想试一下，于是最后也折腾完了，可以实现在本地视频播放器 / 浏览器中播放 PS4 的直播内容，当然不如采集卡好，然而毕竟不花钱，以及还能学点知识。
<!--more-->

### 0. 环境
 - 路由： Openwrt  
 - 系统： Ubuntu  
 - 软件： Nginx with nginx-rtmp-module  

### 1. 编译安装 Nginx

需要编译安装 带 nginx-rtmp-module 模块的 Nginx  
 - Nginx: https://nginx.org/en/download.html
 - nginx-rtmp-module: https://github.com/arut/nginx-rtmp-module  

```
# 准备
sudo apt-get install build-essential libpcre3 libpcre3-dev libssl-dev

# 获取 Nginx 源码
wget https://nginx.org/download/nginx-1.10.2.tar.gz
tar -zxvf nginx-1.10.2.tar.gz
cd nginx-1.10.2

git clone https://github.com/arut/nginx-rtmp-module.git

./configure --prefix=/opt/nginx  --add-module=[$pathto]/nginx-rtmp-module/ --with-http_ssl_module --with-debug

make
sudo make install
```
执行 `./configure` 最后会说明 nginx 的相关路径
```
nginx path prefix: "/opt/nginx"
...
nginx http scgi temporary files: "scgi_temp"
```

### 2. 进行配置
#### 配置 nginx-rtmp-module
复制 rtmp 的 stat.xsl 文件：

```
sudo cp [$pathto]/nginx-rtmp-module/stat.xsl /opt/nginx/html
```

使用 www-data 的文件夹权限：
```
sudo chown -R www-data:www-data /opt/nginx/html
```

修改 Nginx 的配置文件 `/opt/nginx/conf/nginx.conf`，首先将最前面 `#user  nobody;` 修改为 `user  www-data;` 与文件夹权限匹配。
添加 rtmp 内容，在 `http {` 字段前添加 :
```bash
rtmp {
    server {
        # 监听端口
        listen 1935;
        application app {
            live on;
        }
    }
}
```
添加上面的应该也就可以运行了，不过我们需要页面来显示统计信息，`http {}` 中的 `server {}` 添加：
```bash
# rtmp 状态页面
location /stat {
    rtmp_stat all;
    rtmp_stat_stylesheet stat.xsl;
}

location /stat.xsl {
    root /opt/nginx/html;
}
```

执行 `sudo /opt/nginx/sbin/nginx -t` 测试配置文件。  
执行 `sudo /opt/nginx/sbin/nginx` 运行 Nginx ，此时访问 http://127.0.0.1 会显示 Nginx 的欢迎界面， 访问 http://127.0.0.1/stat 会显示 rtmp 的统计页面。

#### 测试  
先用 ffmeng 命令模拟传送一个直播源作为测试 （下面命令推送的视频格式为 H.264， 音频格式为 AAC）:
```
ffmpeg -re -i video.mp4 -vprofile baseline -vcodec copy -acodec copy -strict -2 -f flv rtmp://localhost/app/live
```

刷新统计页面会看到一个名为 live 的 live streams ，用 VLC 或者其他视频播放器选择打开 URL `rtmp://127.0.0.1:1935/app/live` 即可播放了。

#### 劫持 live.twitch.tv
劫持 live.twitch.tv 转发到到本地 Nginx ，听说较旧的 PS4 系统直接在路由器修改对应的 DNS 就可以，现在的系统已经不行了，需要劫持对应 IP 的流量才可以，可以使用 iptables 命令。

查询 ： `nslookup live.twitch.tv`
我这里获得 IP 是 192.108.239.80-86 / 23.160.0.80-86，对应 iptables 命令，在 Openwrt 的路由器上执行：
```bash
iptables -t nat -A PREROUTING -d 192.108.239.0/24 -p tcp --dport 1935 -j DNAT --to-destination xxx.xxx.xxx.xxx:1935
iptables -t nat -A PREROUTING -d 23.160.08.0/24 -p tcp --dport 1935 -j DNAT --to-destination xxx.xxx.xxx.xxx:1935
iptables -t nat -A POSTROUTING -j MASQUERADE
```
我们需要测试劫持是否成功， 再一次执行 ffmpeg 的测试命令，不过将 IP 替换为 Twitch 的 IP 或者域名 `rtmp://live.twitch.tv:1935/app/live` ，然后刷新 rtmp 统计页面，或者直接在播放器仍然打开 `rtmp://127.0.0.1:1935/app/live` ，如果成功，会和之前时一样的效果。  
这时我们就可以打开 PS4 ，进行直播，选择 Twitch ，此时刷新 http://127.0.0.1/stat 我们会看到一个 `live_xxxx_xxxxxxx` 的直播，打开视频播放器将 `rtmp://127.0.0.1:1935/app/live` 中的 `live` 替换为 这个 id 就可以了。

### 3. 最终配置文件与实现浏览器直接播放  
这个只是在本地播放，通过 rtmp 的设置，可以设置自动录像，push到国内的直播站。如果只是自己本地播放 PS4 的话，这样就可以了，但是基本只能使用播放器播放，可以加上 HLS （HTTP Live Stream）支持，可以实现在浏览器内播放，不过 HLS 的延时是要比 rtmp 延时多很多的 （我这边本地直接播放 rtmp 大概在 5s 左右， HLS 的就长多了）。最终我的的设置如下（省略了注释和未修改的 Nginx 配置内容）：

```bash
user  www-data;
worker_processes  1;

events {
    worker_connections  1024;
}

rtmp {
    server {
        # 监听端口
        listen 1935;
        chunk_size 4000;
        # rmtp
        application app {
            live on;
            # 开启 HLS
            hls on;
            # hls 存放视频的文件夹
            hls_path /tmp/hls;
        }
    }
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    # ... server 段落之前内容没有修改  

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        # 添加的内容
        # rtmp 状态页面
        location /stat {
            rtmp_stat all;
            rtmp_stat_stylesheet stat.xsl;
        }
        location /stat.xsl {
            root /opt/nginx/html;
        }

        # HLS 配置 /hls 是访问视频的地址
        location /hls {
            # Serve HLS fragments
            types {
                application/vnd.apple.mpegurl m3u8;
                video/mp2t ts;
            }
            root /tmp;
            add_header Cache-Control no-cache;
        }
        # hls end
        # 添加的内容结束

        location / {
            root   html;
            index  index.html index.htm;
        }

        # .... 余下 Nginx 配置文件内容都未修改
    }
}
```

支持 HLS 的话我们需要视频格式为 H.264 ，音频格式为 AAC ，使用之前的命令推送就可以了。 hls 视频对应的链接是 http://127.0.0.1/hls/live.m3u8  
Chrome 安装 [Native HLS Playback](https://chrome.google.com/webstore/detail/emnphkkblegpebimobpbekeedfgemhof) 直接访问就可以播放了。  
或者下载 [Momovi video hls](https://momovi.com/opensource/momovi-video-hls) 放在 Nginx 的目录, 将 player.html 最后的 url 链接替换成上面链接，然后访问该页面就可以了。  

因为 PS4 的链接很长而且也不是固定的，所以可以将 rtmp 字段修改为下面的, 这样 hls 的地址就是固定的 http://127.0.0.1/hls/live.m3u8 ：

```bash
rtmp {
    server {
        # 监听端口
        listen 1935;
        chunk_size 4000;
        # rmtp
        application app {
            live on;
            # -> hls
            push rtmp://127.0.0.1:1935/hls/live;
        }
        application hls{
            # 开启 HLS
            hls on;
            # hls 存放视频的文件夹
            hls_path /tmp/hls;
        }
    }
}
```

### 4. iptables 规则脚本
可以将 Iptables 命令保存成脚本，便于恢复与重新执行:  
`ps4livestart.sh`：
```bash
#!/bin/bash
iptables -t nat -A PREROUTING -d 192.108.239.0/24 -p tcp --dport 1935 -j DNAT --to-destination xxx.xxx.xxx.xxx:1935
iptables -t nat -A PREROUTING -d 23.160.08.0/24 -p tcp --dport 1935 -j DNAT --to-destination xxx.xxx.xxx.xxx:1935
iptables -t nat -A POSTROUTING -j MASQUERADE
```
`ps4livestop.sh`：
```bash
#!/bin/bash
iptables -t nat -D PREROUTING -d 192.108.239.0/24 -p tcp --dport 1935 -j DNAT --to-destination xxx.xxx.xxx.xxx:1935
iptables -t nat -D PREROUTING -d 23.160.08.0/24 -p tcp --dport 1935 -j DNAT --to-destination xxx.xxx.xxx.xxx:1935
iptables -t nat -D POSTROUTING -j MASQUERADE
```

### 参考链接：
 - [Getting started with nginx rtmp](https://github.com/arut/nginx-rtmp-module)
 - [不用采集卡搭建PS4直播环境-CentOS-7网络设置](https://odinliu.com/2015/01/28/%E4%B8%8D%E7%94%A8%E9%87%87%E9%9B%86%E5%8D%A1%E6%90%AD%E5%BB%BAPS4%E7%9B%B4%E6%92%AD%E7%8E%AF%E5%A2%83-CentOS-7%E7%BD%91%E7%BB%9C%E8%AE%BE%E7%BD%AE/)
 - [如何用最低的成本转播PlayStation 4到国内的直播平台](http://colinzhang.com/2015/10/how-to-push-play-station-4-live-stream-with-lower-cost/)
 - [https://docs.peer5.com/guides/setting-up-hls-live-streaming-server-using-nginx/](https://docs.peer5.com/guides/setting-up-hls-live-streaming-server-using-nginx/)
