---
title: Ubuntu 下通过 Wine 运行日文 Galgame
tags:
  - Ubuntu
  - Wine
  - Galgame
categories:
  - Linux
date: 2016-07-15 23:50:00
---

## 环境

系统版本 ： Ubuntu 16.04 LTS (x64)  
Wine 版本 ： Wine 1.9.14  
Game A ： Cafe Sourire (Install on Windows，Copy to Ubuntu)  
Game B ： Your Diary +H (Install on Ubuntu)  
<!--more-->
## 步骤

### 1. 安装 Wine & Winetricks:

```
# https://wiki.winehq.org/Ubuntu
sudo dpkg --add-architecture i386

sudo add-apt-repository ppa:wine/wine-builds
sudo apt-get update
sudo apt-get install --install-recommends winehq-devel
```
仓库的 wintricks 不好使了，很多文件下载都出了问题，github 上的要好一些
```
git clone https://github.com/Winetricks/winetricks.git
./src/winetricks
```

### 2. 添加日文语言支持

在系统设置 - 语言支持中添加日文  
通过日文环境运行程序的命令为：
```
env LC_ALL=ja_JP.utf-8 wine xxx.exe
```

### 3. 设置 Wine

参考链接： [http://forum.ubuntu.org.cn/viewtopic.php?t=451257](http://forum.ubuntu.org.cn/viewtopic.php?t=451257)

这时可以通过执行 `env LC_ALL=ja_JP.utf-8 wine xxx.exe` 来运行程序，包括安装游戏和运行程序本体，但是游戏无法处理视频内容的播放，在需要播放视频的地方游戏会报错跳出。终端中可以看到 `fixme:quartz:MPEGSplitter_query_accept MPEG-1 system streams not yet supported.` 的内容。

首先创建一个 32 位的 WINEPREFIX （因为执行 `winetricks wmp10` 提示需要在纯 32 位的 wineprefix 下才可以执行），路径中的 `$HOME` 替换为实际的 HOME 路径。

```
export WINEARCH=win32
export WINEPREFIX="$HOME/.wine32";
wineboot -u
```

然后通过 winetricks 安装相关组件
```
winetricks d3dx9
winetricks quartz
winetricks devenum
winetricks wmp10
winetricks gdiplus
```

安装 Your Diary +H : 挂载镜像后执行 `env LC_ALL=ja_JP.utf-8 wine setup.exe`

### 4.快捷方式

安装 Your Diary +H 后，会在桌面创建快捷方式，编辑该文件，在 Exec 字段 WINEPREFIX 前加入 `LC_ALL=ja_JP.utf-8` 否则程序无法正常运行。

```
[Desktop Entry]
Name=your diary +H
Exec=env LC_ALL=ja_JP.utf-8 WINEPREFIX="$HOME/.wine32"; wine C:\\\\Program\\ Files\\\\CUBE\\\\your\\ diary\\ +H\\\\yourdiary+H.exe
Type=Application
StartupNotify=true
Icon=E63C_NewShortcut1_C13072A6768643A6966573D014956572.0
```

然后可以通过命令行的方式 `env LC_ALL=ja_JP.utf-8 wine xxx.exe` 或者快捷方式运行游戏 。

### 5.问题

*   Your Diary +H 1920 x 1080 分辨率花屏， 1600 x 900 倒是正常。两个游戏的全屏模式也都是黑屏，无法正常工作。
*   双显示器时，将已运行的游戏拖动倒另一个显示器之后画面会变得静止不动，拖回来后正常。
*   还有，我并不懂日语。(； ･\`д･´)

---

中间碰到的问题:

搜索的时候，所搜的文章中说到的命令都是 `env LANG=ja_JP.utf-8 wine xxx.exe` ，然而自己执行这个命令时，仅仅 Wine 的 UI 组件更新为了日文，打开后游戏的内容依然是乱码。

自己折腾许久后依然毫无变化，包括添加字体、使用 `ja_JP.ujis` 等等，并无卵用。

然后试了一下 `LC_ALL=ja_JP.ujis` 时，可以正常显示。

尝试一番，仅指定 LANG 是不够的，还要加上 LC_CTYPE 。

然后发现使用 ja_JP.utf-8 也是可以正常显示的 (´・Å・\`)

参考链接： [https://help.ubuntu.com/community/Locale](https://help.ubuntu.com/community/Locale)

> LANG  
> Provides default value for LC_* variables that have not been explicitly set.  
> LC_CTYPE  
> How characters are classified as letters, numbers etc. This determines things like how characters are converted between upper and lower case.  
> LC_ALL  
> Overrides individual LC_* settings: if LC_ALL is set, none of the below have any effect.

除 LANG 与 LC_ALL 之外的 LC_* 分别对应各自的分类属性。  
指定 LANG 时 ： 当 LC_ALL 未指定时，为其他 LC_* 提供默认值，当 LC_* 自身未指定内容时生效。  
指定 LC_ALL 时 ： 忽视 LANG 与 LC_* 自身指定的值，全部取 LC_ALL 所指定的内容 。  

### 日文编码

Shift-JIS : ja_JP.ujis  
utf-8 : ja_JP.utf-8

### locale.gen

列出当前所支持的语言环境 : `locale -a`  
编辑 locale.gen 文件 :  `sudo vim /etc/locale.gen`  
更新 :  `sudo locale-gen`

---

きっと、澄みわたる朝色よりも ：

`env LC_ALL=ja_JP.utf-8 wine asairo.exe`  播放视频内容时报错：

> fixme:gstreamer:unknown_type Could not find a filter for caps: video/mpeg, systemstream=(boolean)true, mpegversion=(int)1
>
> fixme:gstreamer:watch_bus decodebin0: 您的 GStreamer 安装缺少插件。

之前俩使用的都是 quartz ，可以通过运行 `winecfg` 在函数库中停用 `winegstreamer.dll` 解决。

这样之后，能运行起来的多了不少，只是 FAVORITE 家的（比如星空的记忆）总是会在出现对话框时报错跳出，还有个别的播放视频会有点问题，比如闪屏，但不会出错跳出。
