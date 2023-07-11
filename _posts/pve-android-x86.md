---
title: PVE 安装安卓 X86
date: 2023-07-08 15:33:54
tags:
 - PVE
 - Android
categories:
 - Note
---

PCR 更新 4.0 版本后，一直用的 BlueStacks 玩起来有 BUG 了，更新到新的版本结果边栏的广告也太逆天了。尝试了一下其他的模拟器，都有各种各样的毛病
 - 雷电: 为谁炼金商店无法显示，导致频繁弹无法显示购买物品的窗口，也有些卡
 - 夜神: 卡
差点就去买个安卓平板了，但一想摸鱼还是需要在电脑上鼠标操作舒服，多个平板和用手机也没啥区别。

于是打算装一个安卓 x86 试试兼容性如何。

<!--more-->

参考: [在PVE中安装安卓x86](https://foxi.buduanwang.vip/virtualization/pve/567.html/)

## 下载镜像

使用 [清华大学镜像站](https://mirrors.tuna.tsinghua.edu.cn/osdn/android-x86/)

我下载的是 `android-x86_64-9.0-rc2.iso`

## 创建虚拟机

操作系统：
 - 类别：Linux 
 - 版本：6.x-2.6 Kernel
 - 镜像选择对应的 iso
系统：
 - 显卡：SPICE
 - 机型：q35
 - BIOS: OVMF (UEFI)
磁盘： 
 - 总线/设备选择 VirtIO Block

最终配置
![01.png](https://m.nep.me/blog/post/pve-android-01.png)

## 安装
开机后，进入镜像选择，选择 `Advanced options` 选项
![02.png](https://m.nep.me/blog/post/pve-android-02.png)

然后选择 `Android-x86 9.0-rc2 Auto Install to specified harddisk` 进行安装
![03.png](https://m.nep.me/blog/post/pve-android-03.png)

下一步 `Auto Installer` 选择 `Yes`
![04.png](https://m.nep.me/blog/post/pve-android-04.png)

下一步选择 `Run Android-x86`，然后点击ok
![05.png](https://m.nep.me/blog/post/pve-android-05.png)

然后就是安卓开机引导，网络选择跳过，如果先连接了 `VirtWifi` 后续就会卡住，完成引导后，进入系统再连接 VirtWifi 即可

## 设置 ARM 兼容

下载 `houdini.sfs` 文件：http://dl.android-x86.org/houdini/9_y/houdini.sfs

重命名为 `houdini9_y.sfs` 

打开终端，执行下面的命令(先执行 su 获取 `root` 权限)
```
mkdir -p /data/arm
cp /sdcard/houdini9_y.sfs /data/arm/
enable_nativebridge
```
![06.png](https://m.nep.me/blog/post/pve-android-06.png)

打开系统设置 > 安卓X86设置，开启兼容模式

设置完成后，游戏 APP 从打开旧闪退到打开进入首页才闪退了，还是不能玩耍（

白搞。