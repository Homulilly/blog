---
title: WNDR4300 OpenWrt CC 安装 Python3-pip
date: 2016-09-22 18:12:43
categories:
 - Note
tags:
 - OpenWrt
 - WNDR4300
 - Python
---
WNDR4300 的 OpenWrt 15.05.1 的源仓库虽然有 python3 但是没有 python3-pip 和 python3-setuptools, python2.7 的倒是有。  
<s>我又只看了 Python3 的教程</s>  
现在开发版 (trunk) 中已经有了，所以最方便的就是直接用 trunk 版了，但是又碰到别的软件不能用。  

搜索了一下，可以自己编译然后安装的。
最后折腾折腾编译了个 Python3.5 后终于搞定了，下面纯粹是记录一下过程，中间有的步骤觉得不那么靠谱。
<!-- more -->

首先从这里 [Installing pip with get-pip.py](https://pip.pypa.io/en/stable/installing/) 看，安装 pip 还是很简单的，首先安装 python3 ，然后一条命令就够了：
```
curl https://bootstrap.pypa.io/get-pip.py -o - | python3
```
然而执行时报错，sad ，记得是提示 setuptools 无法安装，内容
```
error: invalid Python installation: unable to open /usr/lib/python3.4/config-3.4/Makefile (No such file or directory).
```
看了下确实没有这个文件，包括这个文件夹都没有。  

于是尝试自己去编译一下。平台 Ubuntu 14.04.5 LTS x64 。试了一下 16.04 还是有问题，依赖的问题（

首先下载当前固件对应的 Openwrt SDK： <https://downloads.openwrt.org/chaos_calmer/15.05.1/ar71xx/nand/>  
```
cd OpenWrt-SDK-15.05.1-ar71xx-nand
./script/feeds update -a
./script/feeds install -a

make menuconfig
```
如果 `make menuconfig` 报下面的错误：
```
tmp/.config-package.in:8:warning: ignoring type redefinition of 'PACKAGE_base-files' from 'boolean' to 'tristate'
tmp/.config-package.in:54:warning: ignoring type redefinition of 'PACKAGE_busybox' from 'boolean' to 'tristate'
feeds/base/package/utils/busybox/config/Config.in:818: glob failed: No files found "package/utils/busybox/config/libbb/Config.in"
make: *** [menuconfig] Error 1
```
前面的 warning 选择忽略，主要是 glob failed 这个问题，可以通过 `ln -s ../feeds/base/package/utils package/utils` 解决，参考链接：<https://dev.openwrt.org/ticket/18552>

然后编译 Python3.5 的 ipk 文件：
首先移除当前仓库中的 Python3：
```
./script/feeds uninstall Python3
./script/feeds uninstall Python3-bottle
```
可以通过 `make menuconfig` 看看 Laguages 里面还有没有 Python3 相关的。

Python3.5 的 Makefile 我直接从 OpenWrt 的 github 上下载了
```
git clone https://github.com/openwrt/packages.git openwrt-packages
cp -r ./openwrt-packages/lang/python3* package
make menuconfig
```
通过 `menuconfig` 确认下内容。

然后执行 python3 的编译：
```
make package/python3/compile V=99

#make package/python3-bottle/compile V=99
make package/python3-setuptools/compile V=99
make package/python3-pip/compile V=99
```
编辑时间大多数都是缓慢的在下载，根据后面的情况来看 setuptools 与 pip 不用编译，编了也没用上 (sad  

生成的 ipk 文件在 `bin/ar71xx/packages/base` 目录下，把所有 python3* 的内容通过 sftp 传到 OpenWrt 中，通过 opkg 进行安装，因为有依赖的问题，最好按一定顺序安装，我按照之前 opkg 安装的顺序安装了，首先要安装的 python3-base 和 python3-light ，大概就是下面的顺序，安装第一个 python3-base 后可以通过 python3 试试命令是否可用：
```
opkg install python3-base-
opkg install python3-light-
opkg install python3-pydoc-
opkg install python3-email-
opkg install python3-decimal-
opkg install python3-xml-
opkg install python3-ncurses-
opkg install python3-distutils-
opkg install python3-codecs-
opkg install python3-multiprocessing-
opkg install python3-asyncio-
opkg install python3-ctypes-
opkg install python3-dbm-
opkg install python3-gdbm-
opkg install python3-logging-
opkg install python3-lzma-
opkg install python3-openssl-
opkg install python3-sqlite3-
opkg install python3-unittest-
opkg install python3_3.5.2-
opkg install python3-dev-
opkg install python3-lib2to3-
```
这里并没装 python3-setuptools 与 python3-pip ，因为我装了之后执行命令依然提示 `-ash /usr/bin/pip3 not found` ， 文件在，而且有执行权限，但是还是报错。  
我装了个 bash 进入 bash 后执行命令的错误提示变成了：
```
root@OpenWrt:~/ipk/py3.5# pip3
bash: /usr/bin/pip3: /home/collett/git/OpenWrt-SDK-15.05.1-ar71xx-nand_gcc-4.8-linaro_uClibc-0.9.3: bad interpreter: No such file or directory
```

What？ 一脸问号，为啥路径是我编译的机器的路径？！！！   
本来以为又坑了，不过执行了一下 `curl https://bootstrap.pypa.io/ez_setup.py -o - | python3` ，结果 setuptools 安装上去了 O_O ，然后我就是又试了一下 `curl https://bootstrap.pypa.io/get-pip.py -o - | python3` 结果 wheel  安装成功， pip 的部分则提示：
```
Requirement already up-to-date: pip in /usr/lib/python3.5/site-packages
```
然而我去这里目录里面看，里面是有 pip 的文件夹，但是只有文件夹套文件夹，而且都是空的，大概是之前安装 python3-pip 残留的？于是备份一下再来一次
```
mv pip pip_bak
mv pip-8.1.2-py3.5.egg-info pip-8.1.2-py3.5.egg-info_bak

curl https://bootstrap.pypa.io/get-pip.py -o - | python3
```

ㅇㅅㅇ 装上了。。。
然而试了一下，好卡，导入一个模块都要等几秒，还有为啥我 ROM 直接少了50M ？！ 不过倒是工作倒是正常的。  
<s>等下一个稳定版好了，感觉大概又是白折腾。</s>

需要的空间还可以接受，清理了之后 `空闲空间: 73% (69.70 MB)` ， 最开始的时候 `空闲空间: 97% (92.27 MB)` 。

参考链接：
- OpenWrt Using the SDK: https://wiki.openwrt.org/doc/howto/obtain.firmware.sdk
- Pip Installation: https://pip.pypa.io/en/stable/installing/
- Setuptools and Easyinstall: https://pypi.python.org/pypi/setuptools#using-setuptools-and-easyinstall
- Github OpenWrt Package: https://github.com/openwrt/packages/
