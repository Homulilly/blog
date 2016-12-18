---
title: 安装 Dropbox 与 Nemo 文件管理器
tags:
  - Nemo File Manager
  - Ubuntu
categories:
  - Linux
date: 2016-07-28 18:25:00
---

Dropbox 安装 ：[Install Dropbox on your Linux box](https://www.dropbox.com/install?os=lnx)

可以直接下载 deb 文件进行安装，但是这个只是一个安装器，由于 Wall 的存在，后续的下载无法进行，可以通过这个链接下载文件 [https://www.dropbox.com/download?plat=lnx.x86_64](https://www.dropbox.com/download?plat=lnx.x86_64) 直接解压到 Home 目录下后再运行就可以了。

Ubuntu 自带仓库可以直接安装 Nemo 文件管理器 `sudo apt install nemo` 。
<!--more-->  
但是自带的软件仓库没有 nemo-dropbox 这个软件，所以在 Nemo 中无法显示文件的同步状态。

所以最后选择了通过PPA源安装 : [https://launchpad.net/~webupd8team/+archive/ubuntu/nemo](https://launchpad.net/~webupd8team/+archive/ubuntu/nemo)
```
sudo add-apt-repository ppa:webupd8team/nemo
sudo apt-get update
sudo apt-get install nemo nemo-dropbox
```
然而 nemo-dropbox 时似乎会重新配置一遍 dropbox 的文件，配置过程中还是无法下载配置文件。

我安装时执行了 `export all_proxy / http_proxy` 仍然会卡住，最后显示是无法下载文件的错误，不过这时 Nemo 已经可以显示文件的同步状态了。然而执行 dropbox 命令时会提示未配置 dropbox deamon ，是因为 nemo-dropbox 使用的存储位置与 Dropbox 官方提供的并不一样。  

`sudo vim /usr/bin/dropbox`

找到 `PARENT_DIR = os.path.expanduser("/var/lib/dropbox")` 将括号内容修改为 "~" 即可。

Dropbox 自带的状态图示太丑了，可以直接到这个文件目录替换文件修改。
```
/home/alisa/.dropbox-dist/dropbox-lnx.x86_64-7.3.29/images/emblems
```
设置 Nemo 为默认的文件管理器 以及桌面图标使用 Nemo 的图标
```
gsettings set org.gnome.desktop.background show-desktop-icons false
gsettings set org.nemo.desktop show-desktop-icons true
xdg-mime default nemo.desktop inode/directory application/x-gnome-saved-search
```
恢复为 Nautilus
```
gsettings set org.nemo.desktop show-desktop-icons false  
gsettings set org.gnome.desktop.background show-desktop-icons true
xdg-mime default nautilus.desktop inode/directory application/x-gnome-saved-search
```
然而 Nemo 的文件预览会有一个右和下的阴影，不是很喜欢（
