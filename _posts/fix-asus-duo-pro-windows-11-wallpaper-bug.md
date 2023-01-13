---
title: 解决华硕 DUO PRO 在 Windows 11 下自动更换壁纸的 BUG 
date: 2023-01-13 12:59:43
tags:
 - windows 
 - asus
categories:
 - Note
---

解决步骤： 
1. 壁纸使用纯英文路径    
2. 删除 `C:\ProgramData\ASUS\AsusOLEDShifter\` 与 `C:\users\用户名\AppData\Local\ASUS\ScreenXpert` 下的 `temp.jpg` 或 `temp.png`。
3. 重新设置主屏壁纸  
4. 使用 ScreenPad 的控制栏手动设置一下副屏壁纸  
5. 重启电脑。    

<!--more--> 

我在触发 BUG 前的操作，将壁纸由固定图片修改为了幻灯片。 结果 WIN 11 无法实现主屏幻灯片，副屏固定的操作。  

于是用 ScreenPad 程序切换了副屏的壁纸，系统设置中幻灯片设置失效。  
 
然后就持续的切换为固定的壁纸，秒表一掐，三分钟换一次。  

而且重启后副屏也会更换一次，不过副屏并不是 OLED ，只在开机时换一次。  

一开始还以为是 Windows 11 的 BUG ，禁用了同步等各种操作都不行。  

然后通过注册表找到了历史壁纸的具体路径。  

```
计算机\HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\Wallpapers
```

发现历史壁纸路径是在 `C:\ProgramData\ASUS\AsusOLEDShifter\` 下面，一看 AsusOLEDShifter ，原来是华硕 OLED 屏幕保护程序搞的鬼。里面还有个 `config.ini` ，打开一看，存储的路径还有乱码，这还不支持中文的吗(之前倒是一直没有发现问题，不过还没用过试其他路径下的图片，所以可能这几天可能都没正常运行过？)。  

然后尝试把壁纸更换为不包含中文的路径，然后禁用再启用 OLED 的像素偏移开关。  

好家伙，不行。  

打开  `config.ini` 看了下，路径是更新了，但是旁边的 `temp.jpg` 图片还是卡住的那张，我先手动替换一下试试，等了三分钟，嗯，好使。

重启一下，副屏还是会换，上面那个文件夹里，也没有副屏壁纸相关的文件。不过 `temp.jpg` 已可以自动更新了。

这是我掏出 [Everything](https://www.voidtools.com/zh-cn/) 搜索了 `temp.jpg`，果然， 在 `C:\users\[NAME]\AppData\Local\ASUS\ScreenXpert` 还有一张 `Temp.jpg` 文件，给他删除，然后使用 ScreenPad 重新选择一下副屏壁纸。

重启，完美。


