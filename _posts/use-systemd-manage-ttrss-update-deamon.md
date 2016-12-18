---
title: 使用 Systemd 管理 TTRSS 更新进程
date: 2016-10-29 22:33:48
tags:
 - ttrss
 - systemd
categories:
 - Note
---
话说以前自己 TTRSS 的后台更新是靠 screen 或者 tmux ，虽然感觉麻烦，但是 `it works` ，不过换到 16.04 后，不知道为何更新进程老是挂掉，看了下似乎是内存不足的问题，明明 14.04 的时候完全没啥问题。  

看了下 TTRSS 的 wiki ，更新方法都写的很清楚了： [UpdatingFeeds](https://tt-rss.org/gitlab/fox/tt-rss/wikis/UpdatingFeeds)  

其中使用 Systemd 的应该是很方便，以前没看过这方面的内容，今天就看了看。  

<!--more-->
更新 TTRSS 的话，直接复制 wiki 的内容就可以了。在 `/etc/systemd/system/` 文件夹下创建一个 `ttrss.service` 文件，填入下面的内容：
```
[Unit]
Description=ttrss_backend
After=network.target mysql.service postgresql.service

[Service]
User=www-data
ExecStart=/path/to/tt-rss/update_daemon2.php

[Install]
WantedBy=multi-user.target
```
- 查看输出：`journalctl -u ttrss`
- 开机启动：`sudo systemctl enable/disable ttrss`

---
### About Systemd
#### 相关的路径：
- `/etc/systemd/system/*` - 供系统管理员和用户使用
- `/run/systemd/system/*` - 运行时配置文件
- `/usr/lib/systemd/system/*` - 安装程序使用

#### 基本命令：
- 显示状态：`systemctl status`
- 显示激活的单元：`systemctl / systemctl list-units`


- 控制与状态相关：`systemctl start/stop/reload/status <单元>`
- 开机启动相关：`systemctl is-enabled/enable/disable <单元>`
- 禁用与恢复：`systemctl mask/unmask <单元>`
- 显示帮助：`systemctl help <单元>`
- 重载 Systemd：`systemctl daemon-reload`

#### 单元文件内容：
如前文 ttrss 的示例：
```
[Unit]
Description=ttrss_backend
After=network.target mysql.service postgresql.service

[Service]
User=www-data
ExecStart=/path/to/tt-rss/update_daemon2.php

[Install]
WantedBy=multi-user.target
```
##### `[Unit]` - 记录unit文件的通用信息
- `Description`： 描述内容
- `Requires`： 依赖的服务，若选择单元未启动，则本单元启动失败
- `Wants`： 依赖的服务，但选择单元未启动，对本单元无影响
- `After` / `Before`： 定义启动顺序

##### `[Service]` - 记录 [Service](https://www.freedesktop.org/software/systemd/man/systemd.service.html) 的信息
- `Type`： 服务启动方式  
- `User` / `Group`： 用户/用户组
- `ExecStart` ： 启动时执行的命令
- `ExecStartPre`, `ExecStartPost`:  ExecStart执行前后所调用的命令。
- `ExecStop`: 定义停止服务时所执行的命令，定义服务退出前所做的处理。如果没有指定，使用systemctl stop xxx命令时，服务将立即被终结而不做处理。
- `Restart`: 定义服务何种情况下重启（启动失败，启动超时，进程被终结）。可选选项：no, on-success, on-failure,on-watchdog, on-abort  

##### `[Install]` - 安装信息
- `WantedBy`： 何种情况下，服务被启用。`WantedBy=multi-user.target` （多用户环境下启用）
- `Alias`： 别名

参考链接：
 - [Archwiki - Systemd](https://wiki.archlinux.org/index.php/systemd)
 - [编写systemd下服务脚本](http://blog.csdn.net/fu_wayne/article/details/38018825)
 - [Writing systemd service files](https://patrakov.blogspot.com/2011/01/writing-systemd-service-files.html)
