---
title: 升级Ubuntu 16.04 BETA2 ， BUG 都在
date: 2016-03-27 23:00:00
tags:
  - Ubuntu
categories:
  - Linux
---

自己一直以来 Linux 只用过 Ubuntu ， 因为觉得 Unity 还是不错的，时间长了也就用习惯了。不过并不喜欢 Ubuntu 半年一更的更新政策，所以基本上使用的都是 LTS 版本。  
现在 Ubuntu 14.04 也用了比较长时间了，本来对于 16.04 并不是太着急，觉得正式版出来再装也不迟。但是这一个月来，因为环境有变，在使用上又开始了笔记本 (1366 x 768) + 外接显示器 (1920 x 1080) 的使用方式，然后就碰到了几个 BUG ，很是不爽，但是搜索半天找不到解决的办法，便寄希望于 16.04 了。  
<!--more-->
BUG 情况如下：  
1. 在壁纸模式为缩放，且笔记本屏幕在外接显示器左侧的时候，重启电脑或者切换过 HDMI 以后，外接显示器的壁纸会变得模糊。重新设置一次壁纸，或者切换一次壁纸的模式再换回来，但是显然这样很麻烦。简单一些的办法，就是在显示设置里把外接显示器放在左侧，然而在物理上我是把笔记本放在了左侧，鼠标切换时会比较坑，但是经过一阵子后我已经习惯了一些，感觉还好。  
2. 笔记本屏幕 + 外接显示器的情况下，如果使用 unity-tweak-tool 将 Unity 的顶栏设置为半透明，切换过 HDMI 的话，右侧的那一个屏幕的顶栏会随着鼠标的移动闪烁起来，很痛苦。需要在 unity-tweak-tool 里重新设置一下才可以，不知道什么原因，开源/闭源驱动均试过，还好切换 HDMI 的频率不算高。 (后来发现可以通过 `gsettings set org.compiz.unityshell:/org/compiz/profiles/unity/plugins/unityshell/ panel-opacity 0` 来解决，副屏在切换HDMI后顶栏是全透的，至少不会闪。)   
3. 奇葩的平铺壁纸：  
在第一条，有想到通过双屏壁纸来解决，然而只有 适合宽度 可以，但是我又有时候需要将外接显示器切换给其他设备使用，因此适合宽度在单屏下并不合适，尝试平铺，然而我试了试觉得我无法理解 Ubuntu 是如何去平铺壁纸的: [原图][1] - [平铺示例1][2] - [平铺示例2][3]  (´・ω・\`)  

在 Ubuntu 16.04 BETA2 出来后，我终于是表示受不了啦，决定吃螃蟹。  
去 <mirrors.163.com> 下载了镜像，速度飞快。然而 Ubuntu 14.04 自带的创建 USB 启动器写的U盘，重启后卡光标，没反映。去 Windows 用 UltraISO 写了镜像 提示 gfxboot.c32: not a COM32R Image boot: ，然而镜像 MD5 检验没有问题，换了一下写入方法，都一样，搜索发现  
>This error happens because you used a distribution with an earlier version of the syslinux package to create the bootable USB of a distribution expecting a later version ([bug link][4]).  [via link][5]  

解决方法，很简单，在 boot: 后面输入 live 回车即可。  (ㆆᴗㆆ)   

安装，从 14.04 升级到 16.04 的选项是灰色的，只能全新安装，估计应该是因为 BETA 版的缘故。于是在只保留了 /home 的情况下安装的 Ubuntu 16.04 BETA2 。 然而装完，很遗憾，上面的所说的 BUG 都还在，看来也不指望正式版了，估计是使用环境与情况也是够少见的。  

于是，记记安装软件的流水账，以免以后再重装的时候想不起来有什么需要安装的。  

* apt-get install
>vim  
fcitx fcitx-pinyin fcitx-anthy  
zsh <https://github.com/robbyrussell/oh-my-zsh>  
git-core  
audacious  
smplayer  
tompoy  
gimp  
aria2   
unity-tweak-tool  
indicator-multiload  
tsocks /etc/tsocks.conf  
shadowsocks  
deluge  
synaptic  

* PPA  

Shadowsocks-qt5
```
sudo add-apt-repository ppa:hzwhuang/ss-qt5
sudo apt-get update && sudo apt-get install shadowsocks-qt5
sudo apt-get remove appmenu-qt5 #可以显示菜单栏
```
Numix
```
sudo apt-add-repository ppa:numix/ppa
sudo apt-get update
sudo apt-get install numix-icon-theme-circle  numix-gtk-theme
#apt-get安装的 numix-gtk-theme 有点 BUG ，有像素对不齐的现象。
```

* 需要下载的第三方软件  
>WPS  <http://wps-community.org/downloads/>  
Nylas N1 <https://invite.nylas.com/download/>  
Steam <http://store.steampowered.com/about/>  
Telegram <https://www.telegram.org/dl/tdesktop> #最小化启动：[path]/Telegram -startintray  
BTSync  <https://www.getsync.com/>  
Google Chrome <https://www.google.co.jp/chrome/browser/desktop/>  
#安装Chrome提示依赖关系未解决  `sudo apt-get -f install`

* Change hostname to Ubuntu / 安装时并不能这么做 ( ˘･з･)
```
sudo vim /etc/hostname
sudo vim /etc/hosts
sudo hostname Ubuntu
```

* 其他  
```
#自动挂载某一磁盘  查看 UUID && 编辑 fstab \040表示空格
sudo blkid  
sudo vim /etc/fstab
#安装字体  
Source Code Pro Download：https://github.com/adobe-fonts/source-code-pro/releases
#删除Thunderbird
#禁用Guest账户登陆 add allow-guest=false to 50-ubuntu.conf
sudo vim /usr/share/lightdm/lightdm.conf.d/50-ubuntu.conf
```

* Teamviewer  
最后装了 Chrome WebStore 的 [TeamViewer][6]  
好处：无需安装桌面客户端，界面稍微简洁一些，连接远程的时候不会有额外的窗口陪着。  
缺点：不能预先指定分辨率 / 是否显示壁纸，剪贴板是远程 -> 主机单向。  

随后尝试了一下子 KDE ，不太喜欢。  
随后安装了 unity8-desktop-session-mir 尝试了一把，什么玩意儿的感觉。  


  [1]: https://i.imgur.com/FP2UN4X.png
  [2]: https://i.imgur.com/FsQwayj.png
  [3]: https://i.imgur.com/GPNXAdJ.png
  [4]: https://bugs.launchpad.net/ubuntu/+source/usb-creator/+bug/1325801
  [5]: http://askubuntu.com/questions/486602/ubuntu-14-04-lts-live-usb-boot-error-gfxboot-c32not-a-valid-com32r-image
  [6]: https://chrome.google.com/webstore/detail/teamviewer/oooiobdokpcfdlahlmcddobejikcmkfo
