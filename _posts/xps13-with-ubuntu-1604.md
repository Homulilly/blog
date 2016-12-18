---
title: XPS13 with Ubuntu 16.04
date: 2016-12-08 19:10:38
tags:
 - XPS 13
 - Ubuntu
categories:
 - Note
---
硬件：XPS 13 9350 FHD  
系统：Ubuntu 16.04.1 x64 (4.4.0-53-generic & 4.8.12)  

一直是主要用台机，装了 Ubuntu ，本子装了 WIN ，然而本子毕竟低压 U ，太弱，遂打算换过来，一狠心买了 DA200 ，开搞，然并卵，最后 DA200 并不能在 Ubuntu 上用，而且这 hub 也弱的一比，就 USB 口和网卡可以用，中间 USB-C 口内侧的塑料还碎了， WTF ， Life so hard。 满桌子没找到碎渣，我也不知道是在我手里坏的还是本来就坏的。下面这些就记一下，并没有什么卵用。

哦对 DA200 这货，最后我去 WINDOWS 下面试了一下，然而一开始也不能用，需要装 INTEL 的 TB3 驱动才可以，然而 WIN 下都需要先更新 BIOS 才能装个驱动，当然装完后的确可以用了，所以说坑爹。

<!--more-->

### 显卡驱动
Intel 官方有提供工具 ： [Intel Graphics Update Tool for Linux](https://01.org/linuxgraphics/downloads)。
我装了，然而感觉然并卵，并没有什么区别，本来以为会有可以调整屏幕 RGB 比例的工具，结果并不提供。

顺带记一下： [删除 Ubuntu Intel 官方驱动包并添加 pdadoka PPA](https://plumz.me/archives/4591/)

### Chrome 地址栏闪烁
所以说，装了那个显卡驱动，并没有什么卵用，该在的 BUG 都在，不过这个大概是 Chrome 的 BUG ？

```
sudo vim /usr/share/applications/google-chrome.desktop
```

找到 Exec= 字段，`%U` 前添加 `--disable-gpu-driver-bug-workarounds --enable-native-gpu-memory-buffers` 。

再创建个文件 `sudo nano /usr/share/X11/xorg.conf.d/20-intel.conf`
```
Section "Device"
   Identifier  "Intel Graphics"
   Driver      "intel"
   Option      "AccelMethod"  "sna"
   Option      "TearFree"    "true"
   Option      "DRI"    "3"
EndSection
```
我重启就好了， Chrome 默认勾选使用硬件加速，没再改 `chrome://flags`

via: [Annoying flickering in 16.04 LTS - Chrome](https://askubuntu.com/questions/766725/annoying-flickering-in-16-04-lts-chrome)

### 耳机底噪
运行 `alsamixer` ，找到 `Headphone Mic Boost` （第三个） ，调整成 `[dB gain: 10.00, 10.00]` ，往上加一格就好了。

然而重启后就失效，需要设置一下：
保存配置文件： `alsactl --file ~/.config/asound.state store`
恢复配置文件： `alsactl --file ~/.config/asound.state restore`

把恢复的加入到开机启动就可以了，用 Ubuntu 自带开机启动程序的设置就可以。
