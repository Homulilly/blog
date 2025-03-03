---
title: 使用 UTM 在 macOS 上创建 macOS 15 虚拟机
date: 2024-12-12 19:55:18
tags:
 - macOS
 - UTM
 - VM
categories: Note
---

使用了 VMware Fusion 创建了一个 Windows 11 ARM 虚拟机后，想试试能不能也创建一个 macOS ARM 的虚拟机，毕竟有时候还是需要一个不同的环境，我也没有第二个设备。  

下载了 ipsw 恢复镜像，Fusion 看起来并不支持。 

搜索了一下发现 [Parallels](https://kb.parallels.com/cn/125561) 和免费的 UTM 是支持。

下载使用 UTM 安装了，非常的简单，运行也比较流畅。  

<!--more-->

## 下载 ipsw 格式的 macOS 镜像

访问 https://ipsw.me/product/Mac 通过选择设备获取镜像下载链接，下载上通过 Apple 的服务器下载的。  

推荐下载与宿主机相同版本的 macOS 镜像，我下载的是 15.1.1 。

下载后记得打开终端校验一下文件完整性。

```sh
cd Downloads
openssl sha1 UniversalMac_15.1.1_24B2091_Restore.ipsw 

# 输出需要与网页的相同
SHA1(UniversalMac_15.1.1_24B2091_Restore.ipsw)= f9c870544bc208d9574eb07d8226cd80a77e13e2
```

## 下载安装 UTM

UTM 在 AppStore 是付费的，但是 [官网](https://mac.getutm.app/) 可以免费下载。 

双击下载的 UTM.dmg ，将 UTP.app 拖放到 Application 中。  
![UTM Install](https://m.nep.me/blog/post/utm-install.png)

然后从启动台中打开 UTM ，打开时会有隐私询问，选择打开。

## 创建虚拟机
打开 UTM 后，选择创建虚拟机。   
![UTM Privacy ask](https://m.nep.me/blog/post/utm-app-home.png)

我是安装相同架构的系统，所以自定义选择 **虚拟化** 。  
![](https://m.nep.me/blog/post/utm-create-cpu-type.png)

 - 预配置选择 macOS 12+ 
 - 拖放之前下载的 `.ipsw` 镜像到 **导入 IPSW** 界面中
 - 硬件界面设置内存与 CPU 核心数
 
![](https://m.nep.me/blog/post/utm-create-set-ram-and-cpu.png)
 
 - 设置硬盘大小，默认是 64GB，安装完成后会占用 20GB ，剩余 40GB 用，我设置了 128GB
 - **总结** 界面确认一下信息，点击 **存储**

![](https://m.nep.me/blog/post/utm-create-overview.png)

存储后点击启动，会有抹掉磁盘提示，因为是空的，选择好就可以。  
![](https://m.nep.me/blog/post/utm-create-confirm.png)

等待安装完成，出现苹果 LOGO 后进入欢迎界面引导设置，就安装成功了。

![](https://m.nep.me/blog/post/utm-create-apple-logo.png)

![](https://m.nep.me/blog/post/utm-create-welcome.png)

按照引导依次完成设置： 
 - 选择语言
 - 设置地区
 - 登陆账户可以跳过
 - 同意隐私协议
 - 设置账户与密码
 - 设置时区
 - 选择外观主题

然后即可成功进入系统
![](https://m.nep.me/blog/post/utm-create-done.png)

刚安装的系统占用大约 20GB 左右。   
![](https://m.nep.me/blog/post/utm-create-disk-usage.png)
