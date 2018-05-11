---
title: 重装系统后删除 Cygwin 文件夹
date: 2018-03-18 16:00:00
tags:
categories:
 - Note
---

重装系统后， Cygwin 文件夹不太好删除，右键更改所有者有些麻烦。  
搜了一下，复制粘贴几个命令就可以了。

管理员权限运行 cmd 后， cd 到 cygwin 所在的目录，执行下面的命令即可。

```
takeown /r /d y /f cygwin
icacls cygwin /t /grant Everyone:F
del cygwin
```
