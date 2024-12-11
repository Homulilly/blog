---
title: macOS 上无法访问局域网设备
date: 2024-12-04 10:55:10
tags:
 - macOS
categories:
 - Note
---

一觉起来，VSCode 和 Chrome 都无法访问我的 ALL IN BOOM 了，Chrome 访问提示下面的错误
```js
ERR_ADDRESS_UNREACHABLE
```
检查 ALL IN BOOM 发现并没有关机，在终端内 ping 可以正常得到回应，使用 Windows 设备也可以正常访问。  

搜了一下，类似情况还挺多，前往系统的 **设置** > **隐私与安全** > **本地网络** ，开启对应 APP 的权限。  

不过我的本来就是开启状态的，于是尝试了一下关闭重新打开，然后就恢复了。  

<!--more-->

![macOS Local Network Privacy](https://m.nep.me/blog/post/macos-localnet.png)

