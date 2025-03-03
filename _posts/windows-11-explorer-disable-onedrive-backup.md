---
title: 关闭 Windows 11 资源管理器的 OneDrive 开始备份提示
date: 2025-03-02 11:41:06
tags: 
 - Windows
categories:
 - Note
---

Windows 还是喜欢用本地账户，然而资源管理器到处都是 OneDrive 的备份提示，身世烦人，搜索一下发现可以通过编辑注册表关闭。

按 `Windows+R` 打开运行窗口，输入 `regedit` 打开注册表编辑器。

导航到 
```
计算机\HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\StorageProvider\OneDrive
```

编辑 `StorageProviderKnownFolderSyncInfoSourceFactory` 由 `{07CA83F0-DF06-4E67-89DD-E80924A49512}` 改为 null 即可。

修改后立即生效。

不过这样重启后就又恢复了，需要限制这个注册表的权限。

 - 右键点击左侧的 **OneDrive** , 选择 **权限**
 - 点击 **高级**
 - 点击底部 **禁用继承**
 - 选择 **将已继承的权限转为此对象的显示权限** ，然后应用与确定
 - 把 System 、你的用户、Admiinistrators 的完全控制权限的 **允许** 取消
 - 点击应用即可