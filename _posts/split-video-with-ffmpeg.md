---
title: 使用 FFmpeg 分割视频
date: 2024-03-30 12:40:48
tags:
 - ffmpeg
 - debian
 - smb
 - youtube-dl
categories:
 - Note
---

Debian 安装 FFmpeg
```sh 
apt update
sudo apt install ffmpeg
ffmpeg -version
```

假如想获取原视频 00:01:50 至 00:58:57 之间的部分可以使用下面的命令即可(不转码)
```
ffmpeg -i input.mp4 -ss 00:01:50 -to 00:58:57 -acodec copy -vcodec copy output.mp4
```
- `-ss` 指定开始时间点
- `-to` 指定结束时间点，如果持续到结束则不需要写  
将 `-to` 替换为 `-t` 则是指定持续多长时间，比如从开始 30s 开始的之后 30 分钟   

```
ffmpeg -i input.mp4 -ss 00:00:30 -to 00:30:00 -acodec copy -vcodec copy output.mp4
```

<!--more-->
### 使用 youtube-dl 下载 Youtube 视频

使用 youtube-dl nightly [releases](https://github.com/ytdl-org/ytdl-nightly/releases) 版本

然后使用 `youutbe-dl <url>` 命令可以直接下载 Youtube 的视频了，不过下载的视频可能不是自己想要的版本，加上 `-F` 即可查询可下载版本 

```
youutbe-dl -F <url>

[info] Available formats for -:
format code  extension  resolution note
251          webm       audio only audio_quality_medium  129k , webm_dash container, opus  (48000Hz), 164.06MiB
140          m4a        audio only audio_quality_medium  129k , m4a_dash container, mp4a.40.2 (44100Hz), 164.31MiB
160          mp4        256x144    144p   38k , mp4_dash container, avc1.4d400c, 30fps, video only, 49.20MiB
134          mp4        640x360    360p  119k , mp4_dash container, avc1.4d401e, 30fps, video only, 152.12MiB
136          mp4        1280x720   720p  265k , mp4_dash container, avc1.64001f, 30fps, video only, 337.10MiB
298          mp4        1280x720   720p60  350k , mp4_dash container, avc1.640020, 60fps, video only, 445.38MiB
299          mp4        1920x1080  1080p60  591k , mp4_dash container, avc1.64002a, 60fps, video only, 750.52MiB
18           mp4        640x360    360p  214k , avc1.42001E, 30fps, mp4a.40.2 (44100Hz), 272.76MiB
22           mp4        1280x720   720p  394k , avc1.64001F, 30fps, mp4a.40.2 (44100Hz) (best)
```

如果下载 1080P 可以使用
```
youtbe-dl -f 299 <url>
```

指定代理
```
youtbe-dl --proxy socks5://127.0.0.1:7890 -f 299 <url>
```
指定文件名
```
youtube-dl -f 140 <url> -o audio.m4a
```

使用 ffmpeg 合并音频与视频文件
```
ffmpeg -i video.mp4 -i audio.m4a -acodec copy -vcodec copy output.mp4
```

### Debian 挂载 SMB

安装CIFS客户端  
```
apt install cifs-utils
```

手动挂载
```
mount -t cifs //smb.example.com/home /mnt/smb -o username=$username,password=$password
```

取消挂载
```
umount /mnt/smb
```