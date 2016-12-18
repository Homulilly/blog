---
title: Ubuntu 下 Nautilus 生成 GIF 和 视频的文件的预览图
tags:
  - Ubuntu
categories:
  - Linux
date: 2016-06-19 16:29:00
---

不单当前使用的 Ubuntu 16.04 还是之前用的旧版本， Nautilus 的 GIF 和视频文件的预览都只有默认的主题图标。很不爽，一开始以为是 Nautilus 设置中的一项只有 10MB 文件才生成预览的问题，改成了 4GB ，发现并没有什么区别。

没办法，只能搜索搜索了。
<!--more-->
### 1. 视频文件：

安装所需软件：  
`sudo apt-get install ffmpeg ffmpegthumbnailer`

（原文有提到 gstreamer0.10-ffmpeg ， 16.04 仓库没有这个软件了，发现没装也可以正常使用。）

修改 /usr/share/thumbnailers/totem.thumbnailer 文件  
from this:

    [Thumbnailer Entry]
    TryExec=/usr/bin/totem-video-thumbnailer
    Exec=/usr/bin/totem-video-thumbnailer -s %s %u %o

to this:

    [Thumbnailer Entry]
    TryExec=ffmpegthumbnailer
    Exec=ffmpegthumbnailer -s %s -i %i -o %o -c png -f -t 10

清除缩略图的缓存  
    `rm ~/.thumbnails/normal/*`  
    `rm -r ~/.cache/thumbnails`  

via [askubuntu.com](http://askubuntu.com/questions/2608/nautilus-video-thumbnails-without-totem)  

### 2. GIF 文件 ：  

所需软件  
    `sudo apt-get install imagemagick`

创建 thumbnailer 文件:  
    `sudo vim /usr/share/thumbnailers/gif.thumbnailer`  

加入下面内容：

    [Thumbnailer Entry]
    TryExec=convert
    Exec=convert %i[0] -resize %sx%s %o
    MimeType=image/gif;

清除缩略图的缓存  
`rm -r ~/.cache/thumbnails`  

via [askubuntu.com](http://askubuntu.com/questions/627088/nautilus-not-generating-thumbnails-for-gif-images-in-ubuntu-15-04)  
