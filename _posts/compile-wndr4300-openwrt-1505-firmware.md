---
title: 编译 WNDR4300 的 OpenWrt 固件
date: 2016-09-17 11:17:25
categories: Note
tags:
 - OpenWrt
 - WNDR4300
---

### 前言  
四公主限制太多，分享个图片，支持的两个平台都被认证，自家的消息 APP 又不能给自己发，U盘也是太麻烦。  

于是翻出家底，拿出一直闲置的 WNDR4300 ， 话说 4300 固件还是蛮多的，但是还是偏向于官方或者自己动手，一是比较放心，二是实际自己需要的功能也并不多，OpenWrt 官方虽然有 WNDR4300 的固件，但机器的 128M FLASH 可用只有 32M ，实际剩余也就 13MB 左右，着实是浪费。  
搜索一下，扩充到 128M 也很是简单，自己稍微修改一下配置，然后生成一个固件就可以了。  

编译固件有两种方法，一种是直接下载源码编译，一种是使用 Image Generator 生成。因为是第一次编译 Openwrt 固件，一开始直接就选择了通过源码编译，然后碰到了很多问题， 相对来说使用 Image Generator 更为简单方便一些。  
<!--more-->

关于使用源码编译的问题，一个是我这在 Ubuntu 16.04 上有依赖问题，另一个就是生成的 Kernel 是无法安装仓库中的 kmod 相关的模块的，于是如果需要相关的功能需要在编译的时候一起编译，首先以后使用有不确定性，然后编译耗时也非常的长，使用 Image Generator 生成的镜像是可以安装仓库里的模块与软件的，而且生成也很快，我配置完成后暂时没有发现什么问题。  

### 修改 /root 为 128M  
  这个在两种编译方法是操作是一样的，都是下载完 Openwrt 的源码或是 Image Generator 的相关内容后，编辑 `target/linux/ar71xx/image/Makefile` 文件：  
找到
```
wndr4300_mtdlayout=mtdparts=ar934x-nfc:256k(u-boot)ro,256k(u-boot-env)ro,256k(caldata),512k(pot),2048k(language),512k(config),3072k(traffic_meter),2048k(kernel),23552k(ubi),25600k@0x6c0000(firmware),256k(caldata_backup),-(reserved)
```
将 `23552k(ubi)` 修改为`120832k(ubi)` 、`25600k@0x6c0000(firmware)` 修改为 `122880k@0x6c0000(firmware)` 然后再进行编译相关的操作既可。

### 编译固件  
使用系统 Ubuntu 14.04 LTS x64  

#### 使用 Image Generator 
Image Generator 使用起来简单许多，生成的速度也很快。   
##### 1. 安装依赖  
```
apt-get install subversion build-essential libncurses5-dev zlib1g-dev gawk git ccache gettext libssl-dev xsltproc wget
```

##### 2. 下载 Image Generator  
```
wget https://downloads.openwrt.org/chaos_calmer/15.05.1/ar71xx/nand/OpenWrt-ImageBuilder-15.05.1-ar71xx-nand.Linux-x86_64.tar.bz2
tar -jxvf OpenWrt-ImageBuilder-15.05.1-ar71xx-nand.Linux-x86_64.tar.bz2
cd OpenWrt-ImageBuilder-15.05.1-ar71xx-nand.Linux-x86_64
```
##### 3. 检查  
下载完成后可以通过 `make info` 查看是否包含 `WNDR4300` 的 Profile 以及预装的软件。  
执行下面命令获取官方配置文件预装的软件：
```
echo $(wget -qO - https://downloads.openwrt.org/chaos_calmer/15.05.1/ar71xx/nand/config.diff | sed -ne 's/^CONFIG_PACKAGE_\([a-z0-9-]*\)=y/\1/ip')
```

##### 4. 修改ROM大小

##### 5. 生成  
语法：`make image PROFILE=XXX PACKAGES="pkg1 pkg2 pkg3 -pkg4 -pkg5 -pkg6" FILES=files/`  
其中 `FILES=files/` 表示添加自定义文件，如果没有可以直接不写，否则反而会报错 `cannot found files/*` ，PACKAGES 中的 `pkg` 表示添加，`-pkg` 则表示删除。  
最后我使用的：
```
make image PROFILE=WNDR4300 PACKAGES="libiwinfo-lua liblua libubus-lua libuci-lua lua luci luci-app-firewall luci-base luci-lib-ip luci-lib-nixio luci-mod-admin-full luci-proto-ipv6 luci-proto-ppp luci-theme-bootstrap luci-theme-openwrt rpcd uhttpd uhttpd-mod-ubus luci-i18n-base-zh-cn luci-app-transmission luci-app-vnstat luci-app-commands -dnsmasq dnsmasq-full openssh-sftp-server ipset wget aria2 ip iptables-mod-tproxy bind-dig ip iptables-mod-tproxy tcpdump ntfs-3g screen whereis"
```

#### 下载源码编译
很明显的缺点就是无法安装 kmod 依赖，编译时没有编译相关依赖会导致在线仓库中的某些软件无法安装。  

##### 1. 安装依赖包
```
sudo apt-get install subversion build-essential libncurses5-dev zlib1g-dev gawk git ccache gettext libssl-dev xsltproc
```
我编译过程中碰到了缺少 perl xml::parser 的情况导致安装失败，可以安装 `libxml-parser-perl` 。
##### 2. 下载源码
```
git clone -b chaos_calmer https://github.com/openwrt/openwrt.git
cd openwrt.git
./scripts/feeds update -a
./scripts/feeds install luci
```
luci 只是其中的一部分， `./scripts/feeds install -a` 安装全部，但是肯定非常耗时， 也可以 `./scripts/feeds install <packagename>` 单独安装某一软件包，如 `ntfs-3g` 。  

##### 3. 修改ROM大小

##### 4. 编译  
获取官方配置
```
wget https://downloads.openwrt.org/chaos_calmer/15.05/ar71xx/nand/config.diff
```
将 config.diff 文件中的 `CONFIG_TARGET_ar71xx_nand_R6100=y` 行修改为  
```
CONFIG_TARGET_ar71xx_nand_WNDR4300=y
```
生成配置
```
cat config.diff >> .config
make defconfig
```

编译之前，可以通过 `make menuconfig` 检查或是修改预装软件包。确保 `ipset` 或是其他自己需要的软件被勾选。  
执行编译：
```
make
```

##### 5. Ubuntu 16.04 中的问题  
我在 Ubuntu 16.04 中碰到了依赖相关的问题，明明已安装了 `libncurses5-dev libssl-dev zlib1g-dev` 执行 `make defconfig` 依然提示：
```
Build dependency: Please install ncurses. (Missing libncurses.so or ncurses.h)
Build dependency: Please install zlib. (Missing libz.so or zlib.h)
Build dependency: Please install the openssl library (with development headers)
```
搜索并未发现可以通过 apt 解决，自行编译这三个相关的软件应该是可以解决的，然而我选择了虚拟机 14.04 走起。

### 刷机
#### 1. 固件文件
生成的固件均位于 `bin/ar71xx/` 目录下，与 4300 相关的两个文件为：  
```
openwrt-15.05.1-ar71xx-nand-wndr4300-ubi-factory.img
openwrt-15.05.1-ar71xx-nand-wndr4300-squashfs-sysupgrade.tar
```
`ubi-factory.img` 用于 tftp 刷机， `squashfs-sysupgrade.tar` 用于兼容固件直接后台升级。

#### 2. tftp 刷机
```
sudo apt-get install tftp
```

设置电脑 IP 为 192.168.1.2 / 掩码 255.255.255.0 / 网关 192.168.1.1 ，网线插在 LAN 口。  
拔掉电源，戳住 reset ，插电，等待绿灯长亮后可以松开。  
待 192.168.1.1 可以 ping 通后执行下面命令：  
```
echo -e "binary\nrexmt 1\ntimeout 60\ntrace\nput openwrt-ar71xx-nand-wndr4300-ubi-factory.img\n" | tftp 192.168.1.1
```
等待重启即可。

> tftp 上传命令后来发现使用 curl 即可， `curl -T firmware.img tftp://192.168.1.1` 不过还未实验，记录一下。

#### 3. opkg update   
如果使用 Image Generator 生成的固件，执行 `opkg update` 没有源, 先查看 `/etc/opkg/distfeeds.conf` 是否存在，可以在该文件或是 `/etc/opkg.config` 末尾内加入下面内容：
```
src/gz chaos_calmer_base http://downloads.openwrt.org/chaos_calmer/15.05.1/ar71xx/nand/packages/base
src/gz chaos_calmer_luci http://downloads.openwrt.org/chaos_calmer/15.05.1/ar71xx/nand/packages/luci
src/gz chaos_calmer_packages http://downloads.openwrt.org/chaos_calmer/15.05.1/ar71xx/nand/packages/packages
src/gz chaos_calmer_routing http://downloads.openwrt.org/chaos_calmer/15.05.1/ar71xx/nand/packages/routing
src/gz chaos_calmer_telephony http://downloads.openwrt.org/chaos_calmer/15.05.1/ar71xx/nand/packages/telephony
src/gz chaos_calmer_management http://downloads.openwrt.org/chaos_calmer/15.05.1/ar71xx/nand/packages/management
```

### 其他
DNSMasq / ipset / iptables

/etc/dnsmasq.d/custom.conf
```
server=xxx.xxx.xxx.xxx
server=/example.com/xxx.xxx.xxx.xxx#53
ipset=/example.com/redir
```

`ipset -L redir` to check

Add bellow to `Firewall - Custom Rules`:
```
ipset -N redir iphash
iptables -t nat -A PREROUTING -p tcp -m set --match-set redir dst -j REDIRECT --to-port 7777
```
7777 the port of ss-redir

### 参考与相关链接
- [WNDR4300 OpenWrt 固件与 Image Generator 下载](https://downloads.openwrt.org/chaos_calmer/15.05.1/ar71xx/nand/)
- [OpenWrt Image Generator - EN](https://wiki.openwrt.org/doc/howto/obtain.firmware.generate)
- [编译wndr4300 openwrt 15.05固件](https://www.xdty.org/1915)
- [网件Netgear WNDR4300刷OpenWrt](https://github.com/softwaredownload/openwrt-fanqiang/tree/master/ebook/wndr4300)
