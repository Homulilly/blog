---
title: Ubuntu 16.04 更换鼠标样式并修改大小
tags:
  - Ubuntu
categories:
  - Linux
date: 2016-06-19 18:04:00
---

觉得 Ubuntu 自带的鼠标一般般，不是很好看，之前尝试 KDE 时，觉得那个鼠标很不错。  

今天意外的发现有人移植了： [Breeze Serie for Righties](https://www.gnome-look.org/p/999991/)

下载后放到 ~/icons 文件夹下就可以了，然后用 unity-tweak-tool 就可以修改  
`sudo apt-get install unity-tweak-tool`
<!--more-->
然而换完发现图标大小比默认的大了一圈，不喜欢。  
可以用 dconf-editor 修改 /org/gnome/desktop/interface中的 cursor-size

或者直接在终端执行一下  
`gsettings set org.gnome.desktop.interface cursor-size 16`

就可以修改和默认的鼠标一个大小了。

但是注销与重启后会失效，中间尝试了各种办法都没有效果。  
后来试了几次，发现大概是每次登录后2s左右鼠标会被重新设置为当前所选择的鼠标，于是大小也就被重置了。  
所以尝试了在脚本中加入几秒的延时后，果然可以了。  

创建 setcursorsize.sh  文件  

    #!/bin/sh
    sleep 5s
    gsettings set org.gnome.desktop.interface cursor-size 16
    exit

然后 Ubuntu 直接用 “启动应用程序” 创建一下启动项就可以了。  

* * *

然而鼠标的大小依然经常会被重置，再搜索发现另外有默认大小和自带是一致的：[Brezze Snow Cursor Theme](https://aur.archlinux.org/packages/breeze-snow-cursor-theme/)  

真是开心，终于不用强迫症了。
