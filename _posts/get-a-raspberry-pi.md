---
title: 买了个树莓派
date: 2017-01-18 17:05:28
tags:
 - Raspberry
categories:
 - Note
---
本来是去京东长草了魔蛋的 68 键键盘，给了张 299-50 的券，心动了一下，于是犹豫要不要买。
但是感觉用处又不大，自己也很少带出去用，纠结了一会后，然后不知道为何突然想到花 300 块买个键盘不如买个树莓派了，于是我就买了。

现在蛮便宜的了，本体只要 185 ， E14 版本的，然后还稍带了很多配件，最后带邮费花了 260 左右吧。以前关注过，不过一直没买，觉得自己拿来没啥用，不过现在有些需要一个 `Always Online` 的设备，自己的电脑这么用毕竟功耗还是不低的，也就拿来存点配置，然后跑一下小脚本，树莓派感觉够用了，至于之前用的路由器，毕竟性能实在太着急。

Ubuntu下，配置起来也是很简单。  
<!--more-->
我觉得官方支持的 Raspbian 基于 Debian 就挺好。    
下载系统： https://www.raspberrypi.org/downloads/raspbian/  
安装说明： https://www.raspberrypi.org/documentation/installation/installing-images/linux.md  

下载完成后解压，安装到 tf 卡 (/dev/sde) 中 dd 就可以了。  
```
sudo dd bs=4M if=2017-01-11-raspbian-jessie.img of=/dev/sde
```
dd 完后直接插到树莓派中，插电启动就可以了。因为忘记买卡，U盘也突然失踪了，只好暂时先用一张 8G 的 C4 卡顶着，感觉居然能用，启动也大概就在 20s 左右。不过写入只有 2MB/s ，该换还是得换ww  

系统默认用户名是 `pi` ， 密码是 `raspberry` ，我是搜了一下才知道，他们说第一次开机会运行设置向导，然而我并没见到，直接让我登录了呃。 `sudo raspi-config` 可以运行设置向导，里面可以设置扩展 root 分区到整个 SD 卡、启动选项、时区键盘等很多我们需要设置的东西。  

<s>有 HDMI 的设备都是好设备，</s> 可以直接插网线进行连接，不过我没多余的网线了，只能先配置一下 WIFI 来用，可以通过 `sudo iwlist scan` 来扫描 WIFI 列表，然而那屏幕显示... `>_>` ，反正我自己的 WIFI 肯定信息都知道：
```
# 编辑 WIFI 文件
sudo vim /etc/wpa_supplicant/wpa_supplicant.conf
# 在该文件最后添加下面的内容
network={
  ssid="WIFINAME"
  psk="password"
}

# 查看信息
ifconfig wlan0

# 如果等一段时间还没生效
sudo ifdown wlan0 && sudo ifup wlan0
```

安装软件什么的和普通的 Debian 一样使用 apt：
```
sudo apt update && sudo apt upgrade  
sudo apt install zsh git git-core tmux curl tsocks htop aria2
sudo apt python3 python3-pip python-pip
sudo apt transmission-common transmission-cli transmission-daemon
```
除了性能着急，第一次见到百分比是一个数字一个数字的走的 （  

之后买的 tf 卡到了，因为新买的容量大于 8G ，直接 dd 就可以，我只有一个读卡器，只好中转一下。
```
dd bs=4M if=/dev/sde of=./raspbak.img
dd bs=4M if=./raspbak.img of=/dev/sde
```
然后重新运行一下 `sudo raspi-config` 扩展一下 root 分区。

修改 `/boot/config.txt` 可以更改一些设置：
```
# modify
# audio ouput from HDMI
# hdmi_drive=2
hdmi_group=2
# 1920x1080 60Hz
hdmi_mode=82

# for sdcard
dtparam=sd_overclock=100
# modify end
```

然后我觉得就可以把 Hexo 的本地内容放到树莓派上了，首先要安装 Nodejs ，很好的是官方已经提供了 arm 的编译包：
```
curl -O https://nodejs.org/dist/latest-v6.x/node-v6.9.4-linux-armv7l.tar.gz
tar -zxvf node-v6.9.4-linux-armv7l.tar.gz
```
然后把 nodejs 的 bin 目录加入 PATH 就可以用了。

性能嘛。总之 `it works`。
```
INFO  Files loaded in 5.81 s
INFO  199 files generated in 28 s
# vs
INFO  Files loaded in 874 ms
INFO  199 files generated in 3.31 s
```

 <s>要说性能还是 Intel 的 NUC 好。</s> 哦对，还有那啥买的那个小风扇，没想到挺呼啸的，用了一下后我就拆了下来，毕竟现在也才是冬天。


参考：
 - [树莓派3命令行配置wifi无线连接和蓝牙连接](https://www.embbnux.com/2016/04/10/raspberry_pi_3_wifi_and_bluetooth_setting_on_console/)
 - [树莓派连接WiFi（最稳定的方法）](https://i.cmgine.net/archives/11053.html)
 - [树莓派3如何设置HDMI分辨率](http://jingyan.baidu.com/article/22fe7ced2bc4103002617f9f.html)
 - [使用 vcgencmd 指令查看 Raspberry Pi 的信息](https://blog.gtwang.org/iot/raspberry-pi-vcgencmd-hardware-information/)
 - [树莓派3的使用(Raspbian)](https://www.zybuluo.com/yangxuan/note/321467)
 - [树莓派上的Node.js](https://cnodejs.org/topic/54032efa9769c2e93797cd06)
