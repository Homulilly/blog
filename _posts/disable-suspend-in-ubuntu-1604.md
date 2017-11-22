---
title: 禁用 Ubuntu 16.04 睡眠功能
date: 2017-03-22 20:35:47
tags:
 - Ubuntu
categories:
 - Note
---

捡了一台双路 X5650 的电脑，内存是真大，贼爽，就是不能睡眠，睡了之后图形界面起不来，虽然卸载掉闭源显卡驱动后能看到界面了，但是还是卡死状态， ssh 倒是能连上，反正搞了半天没搞好，之后有不小心点了几次睡眠，索性搜了一下禁用睡眠功能的方法。

创建文件 `/etc/polkit-1/localauthority/50-local.d/com.ubuntu.disable-suspend.pkla`， 加入下面的内容，然后重启就可以了：
```
[Disable suspend (upower)]
Identity=unix-user:*
Action=org.freedesktop.upower.suspend
ResultActive=no
ResultInactive=no
ResultAny=no

[Disable suspend (logind)]
Identity=unix-user:*
Action=org.freedesktop.login1.suspend
ResultActive=no
ResultInactive=no
ResultAny=no

[Disable suspend when others are logged in (logind)]
Identity=unix-user:*
Action=org.freedesktop.login1.suspend-multiple-sessions
ResultActive=no
ResultInactive=no
ResultAny=no
```

参考： [How to disable suspend in 14.04](https://askubuntu.com/questions/452908/how-to-disable-suspend-in-14-04)

---
<!--more-->
在登录界面隐藏用户：

在 `/var/lib/AccountsService/users/` 文件夹下创建与用户同名的文件
加入下面的内容:
```
[User]
SystemAccount=true
```

如果文件已存在，把 `SystemAccount=true` 加入 `[User]` 段落。
