---
title: 禁用 Cortana
date: 2018-03-18 15:53:52
tags:
categories:
 - Note
---

打开注册表编辑器 （ Win + X , 运行 regedit ）。

定位至 `HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Windows\`。

如果 Windows 项下没有 Windows Search 子项，则新建一个。

选中Windows Search项，在右侧窗格中点击右键，选择“新建 - DWORD（32位）值”，然后把新建的值命名为AllowCortana，数值数据按默认的 0 即可。

重启Windows资源管理器后生效。

参考：[如何彻底禁用 Cortana](https://www.zhihu.com/question/33787066)


