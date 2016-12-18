---
title: 使用 Youtube-dl 下载 Youtube 视频
date: 2016-09-19 06:36:00
tags:
 - Youtube-dl
categories:
 - Note
---
有时看到一些 Youtube 上的视频还是打算下载下来的，以前觉得省事直接搜索一些网站下载，记不住网址，然后下载也不一定快。  
去 Github 搜索一下有没有什么工具可以使用，搜索到了一个 Youtube-dl-gui ，并不是太好使，这个项目说到使用了 youtube-dl ，看了一下着实有相见恨晚的感觉。使用很方便，功能也很强大。  

项目地址：<https://rg3.github.io/youtube-dl/>  

Ubuntu 上使用 Python pip 安装就可以了，安装命令：
```
sudo pip install youtube-dl
```
或是 （需要将 $HOME/.local/bin 加入 PATH）
```
pip install --user youtube-dl
```
其他平台可以参考 > [安装指南](https://github.com/rg3/youtube-dl/blob/master/README.md#installation) 。   
<!--more-->

使用起来也是很简单，打开终端执行 `youtube-dl [option] $video-link` 即可。

有时选项太多，可以创建一份 `$HOME/.config/youtube-dl/config` 作为默认的配置文件，每次下载 youtube-dl 都会先读取此配置，可以使用 `--ignore-config` 忽略，每个选项一行，如：  
```
# Lines starting with # are comments
# Use this proxy
# --proxy 127.0.0.1:8080
--proxy socks5://127.0.0.1:1080/

# Save all videos under Movies directory in specified directory
-o ~/Downloads/Youtube/%(title)s.%(ext)s
```

youtube-dl 默认会下载最佳质量的视频和音频，并进行合并，可以通过 `-f` 指定视频格式：
```
youtube-dl -f 'bestvideo[ext=mp4]+bestaudio' $video-link
```
视频格式的选择也是相当强大，具体的使用可以参考 > [Format selection examples](https://github.com/rg3/youtube-dl/blob/master/README.md#format-selection-examples) .   

下载的时候也不一定需要输入全网址，输入网址中的视频 id 即可，比如下面的两个命令是等效的：
```
youtube-dl -- cinwXWDOq2k
youtube-dl https://www.youtube.com/watch?v=cinwXWDOq2k
```

总之上面这些简单的用法对于我来说已经够用，详细的用法还是看 [文档](https://github.com/rg3/youtube-dl/blob/master/README.md#readme) 来得实在。

---
Update:
Install avconv or ffmpeg on Ubuntu 14.04:
- avconv
```
sudo apt-get install libav-tools
```
- ffmpeg
```
sudo add-apt-repository ppa:mc3man/trusty-media
sudo apt-get update
sudo apt-get install ffmpeg gstreamer0.10-ffmpeg
```
