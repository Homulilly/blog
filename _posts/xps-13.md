---
title: XPS 13
tags:
  - XPS 13
  - DELL
categories:
  - Just-buy
date: 2016-06-02 22:00:00
---

自己笔记本的念头也有五六年之久了。一直也产生了更换的念头，可惜现在的 VAIO 并没有多少信仰了。

也感觉苹果的本子比较昂贵，Win 阵营中也没有发现比较让人惊艳的。

不过后来发现了 DELL 的 XPS 系列。自己看图片觉得那无边框下面有些丑所以一开始并没有来电，但是后来被默默种了草，成了优先考虑的范围，不过和苹果一样，感觉还是贵。虽然一直海淘价格不错，但是一般都是 Ebay / 和官翻，自己没有转运方面的经验，我也没有 VISA/MasterCard 的卡。

前几天，终于奶茶东上有 XPS 的特价了， XPS 13-9350-R1708 (i7-6500U/8G/256G)，而且可以 12/24 期免息。虽然不是最新的 i7-6560U ，所以内置的显卡 HD520 要比 Iris 540 弱非常多。 犹豫了半天，还是趁着免息下了了单。
<!--more-->
随后到手，开箱A面的铝合金看着还是比较好看的，不过仔细看的话会发现上面会有一些黑点，小坑类的，不多，有的可以擦掉，有的就。。那样了。

外观，A面铝合金，C面感觉又是啥类肤材质，感觉还好，不会出现过电的现象。

接口，两个 USB3.0 ，一个 Thunderblot 3 ，一个SD读卡器，没有HDMI很遗憾，虽然 Thunderblot 3 可以转，毕竟是额外支出，搜索的话配件也都是沾了 Apple 的光，而且转接器再小巧也不方便。

最大的亮点，屏幕，第一眼，以前的感觉是正确的，果然下方那么宽导致有点丑，不过看得时间长一些，就是越看越好看。

然后折腾系统，重新安装 Win 10 ,直接插 U 盘，在安装时会提示找不到硬盘，所以需要提前准备下面两个东西，或者可以参照这个 [Guide](https://www.reddit.com/r/Dell/comments/3sr1jh/windows_10_clean_install_guide/)

*   下载 Intel® RST 驱动，在安装的时候手动加载：[https://downloadcenter.intel.com/download/25165/Intel-Rapid-Storage-Technology-Intel-RST-RAID-Driver](https://downloadcenter.intel.com/download/25165/Intel-Rapid-Storage-Technology-Intel-RST-RAID-Driver)
*   网卡驱动：[http://www.dell.com/support/home/cn/zh/cndhs1/product-support/product/xps-13-9350-laptop/drivers](http://www.dell.com/support/home/cn/zh/cndhs1/product-support/product/xps-13-9350-laptop/drivers)
> 关于 Intel® RST 驱动，后来感觉应该在 BIOS 里的 SATA Mode ，将默认的 Raid ON 改为 Disable 应该就可以了，不过我并没有测试。

重装完，硬盘为三星 PM951，磁盘写入速度一般 [CrystalDiskMark 结果](https://i.imgur.com/AyDjZQhl.png) ，但是 CrystalDiskInfo 提示找不到硬盘，第一次见到，网上搜索一圈发现，卸载 irst 驱动，然后在 BIOS 里将 SATA Mode 设为 Disable， 两个操作少一个都会无法进入系统，卸载后并没有发现什么区别，不知道这个驱动是做什么的（所以说装系统时一开始就在 BIOS 里将 SATA 设为 Disable 应该就可以了。

> 卸载 Irst 驱动：  
> 设备管理器 - 存储控制器 INTEL SATA RAID Controller﻿  
> ﻿然后重启在BIOS中 将SATA Mode 设为 Disable  
> 卸载后会变成 标准NVM Express控制器  

Win 10下使用，默认的放大倍数为150%，除了部分图片APP感觉有些虚以外都很好。

然而一开始安装 10240 版本，不知道为什么无法更新，报错 0x80240fff , 网上方法试了都不行，最后重装才可以。

关于触摸板，以前摸过 Mac 的触摸板确实好用，一开始的感觉，按压的手感不好， 随后感叹了一下 还是 Apple 的触摸板好用，但是随后发现，使用过程中基本用不倒按压这一操作，轻触就可以，所以说触摸板还是非常好使用的，灵敏度不错。

<del>出于信仰</del>，打算安装 Ubuntu 试试，因为已经搞定了 卸载 Irst 驱动，关闭 SATA Mode ，安装过程中完全没有问题，在最新的 16.04 LTS 版本中，驱动完全已经无需自己安装，包括无线网卡的驱动。

然而 256GB 的磁盘，在安装双系统的时候就非常着急了，看了下 512G 的价格，我选择忍了。

总体来说 13 寸下的 1080P 无论再 Win 还是Ubuntu下 都需要调一下放大倍数， 不然都觉得太小，但是还是感觉 Ubuntu 稍微更加讨喜一些，毕竟 Win 下没有优化的程序会有糊一脸的感觉。而且这本子，毕竟低压U，性能有限，定位也是办公/便携需求，卖的也就是外观，就性价比来说，其实一般。感觉重量 1kg+ 还是有点沉，那些 700g 的黑科技感觉一定非常不错。

* * *

2016.06.10

Ubuntu 16.04 LTS 在 Kernel 4.4 下显卡驱动有问题，会闪屏，升级到 4.6 内核就可以了。

升级后重启 WIFI 可能会无法使用， 手动执行一下 `sudo service network-manager restart` 或者再重启一下就好了。
