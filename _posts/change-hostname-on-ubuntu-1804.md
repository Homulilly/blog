---
title: Ubuntu 18.04 修改 Hostname
date: 2018-05-11 09:44:50
tags: 
 - Ubuntu
categories:
 - Note
---

装了 Ubuntu 18.04， 用 `hostnamectl set-hostname <Hostname>` 修改了 Hostname， 结果重启后又变了回来。

搜了一下需要在文件 `/etc/cloud/cloud.cfg` 中将 
```
preserve_hostname: false
```
修改为
```
preserve_hostname: true
```

然后再重启系统就可以了。
