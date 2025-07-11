---
title: 删除 ColorOS 预装 APP 
date: 2022-12-31 14:53:19
tags:
 - ColorOS
categories:
 - Note
---

上月底，一加 7T Pro 的安卓 12 更新终于来了，也由氢OS 变成了 ColorOS，界面上变化很大。

如果不是有安卓 12 的大版本更新，提不起更新的动力，毕竟很多东西已经使用习惯了，不过更新后用了一个多月了，感觉还行，起码大毛病没有，小毛病有一些，流畅度上没有问题，一种 855Plus 还能再战两年的感觉。

但是很多 ColorOS 的自带 APP 不能卸载，也不能在设置里停用，需要掏出 ADB 。

论坛里有人做了包名整合帖子，这里摘抄一下。

<!--more-->

本文对应的是 `ColorOS 12`

## ADB Tools

Google 官方 的 ADB 工具 Platform-Tools 下载地址 : [https://developer.android.com/tools/releases/platform-tools?hl=zh-cn](https://developer.android.com/tools/releases/platform-tools?hl=zh-cn)  

[百度盘](https://pan.baidu.com/s/1yMR4C6s30h34DJHQaUOjxw?pwd=v8vx) 
```
名称: platform-tools-latest-windows.zip
版本：36.0.0
大小: 7138784 字节 : 6971 KiB
SHA1: 18bb505f9fbfbdf1e44fca4d794e74e01b63d30e
```

先解压文件夹，Windows 使用资源管理器打开解压后的文件夹，在地址栏中输入 `cmd` 或是 `powershell` 即可快速打开终端界面。  


## 开启 USB 调试
ColorOS 开启 USB 调试的方法  
 - 打开 `设置` > 底部 `关于本机` > 打开 `版本信息`
 - 多次点击版本号，输入锁屏密码解锁 `开发者选项`
 - 打开 `设置` > 底部 `系统设置` 或是 `系统与更新` > 打开 `开发者选项`  
 - 找到 `调试` > 开启 `USB 调试`  

手机开启 USB 调试，然后连接电脑，模式选择传输文件，在手机上选择 `允许 USB调试`，然后执行下面命令查看连接设备  
```sh
# 查看连接设备
adb devices

# 成功的结果输出
List of devices attached
xxxxxxx        device
```

## ADB 基础操作
```sh
#删除
adb shell pm uninstall -k --user 0 [Package Name]

#停用
adb shell pm disable-user [Package Name]

#启用
adb shell pm enable-user [Package Name]

#恢复 
adb shell pm install-existing --user 0 [Package Name]

#查看应用列表
adb shell pm list package [Package Name]
```

## 删除 ColorOS 自带 APP 

停用速览
```sh
adb shell pm disable-user com.coloros.assistantscreen
```

停用手机管家（影响：杀毒无，垃圾清理无，安全支付不显示，安全事件不显示）
```sh
adb shell pm disable-user com.coloros.phonemanager
```
删除手机管家（影响：需要用指令恢复安装）
```sh
adb shell pm uninstall -k --user 0 com.coloros.phonemanager
```

停用支付保护（影响：没有支付安全提示）
```sh
adb shell pm disable-user com.coloros.securepay
```

停用数据服务平台
```sh
adb shell pm disable-user com.coloros.sceneservice
```

停用健康数据平台（运动健康app服务）
```sh
adb shell pm disable-user com.oplus.healthservice
```

停用融合搜索服务
```sh
adb shell pm disable-user com.oplus.dmp
```

删除快应用
```sh
adb shell pm uninstall -k --user 0 com.nearme.instant.platform
```

删除移动服务
```sh
adb shell pm uninstall -k --user 0 com.heytap.htms
```

删除自带浏览器，新版本系统不支持卸载和停用了
```sh
adb shell pm uninstall -k --user 0 com.heytap.browser
```

删除钱包
```sh
adb shell pm uninstall -k --user 0 com.finshell.wallet
```

删除 Oppo 后台广告
```sh
adb shell pm uninstall -k --user 0 com.opos.ads
```

删除语音助手
```sh
adb shell pm uninstall -k --user 0 com.heytap.speechassist
```

删除删除 Color 视频
```sh
adb shell pm uninstall -k --user 0 com.heytap.yoli
```

删除乐滑锁屏
```sh
adb shell pm uninstall -k --user 0 com.heytap.pictorial
```

删除用户体验计划
```sh
adb shell pm uninstall -k --user 0 com.oplus.statistics.rom
```

删除 Color 音乐
```sh
adb shell pm uninstall -k --user 0 com.heytap.music
```

删除设备快连
```sh
adb shell pm uninstall -k --user 0 com.heytap.accessory
```

删除儿童空间
```sh
adb shell pm uninstall -k --user 0 com.coloros.childrenspace
```

删除耳机返听
```sh
adb shell pm uninstall -k --user 0 com.coloros.karaoke
```

删除远程守护服务（影响：color系手机远程协助不能用）
```sh
adb shell pm uninstall -k --user 0 com.coloros.remoteguardservice
```
停用
```sh
adb shell pm disable-user com.coloros.remoteguardservice
```

删除跨屏互联
```sh
adb shell pm uninstall -k --user 0 com.oplus.synergy
```

删除小布扫一扫
```sh
adb shell pm uninstall -k --user 0 com.coloros.ocrscanner
```

删除小布扫一扫服务后台
```sh
adb shell pm uninstall -k --user 0 com.coloros.colordirectservice
```

删除小布识屏
```sh
adb shell pm uninstall -k --user 0 com.coloros.karaokedirectui
adb shell pm uninstall -k --user 0 com.coloros.directui
```

删除安全事件
```sh
adb shell pm uninstall -k --user 0 com.coloros.securityguard
```

删除全局搜索
```sh
adb shell pm uninstall -k --user 0 com.heytap.quicksearchbox
```

删除智能驾驶
```sh
adb shell pm uninstall -k --user 0 com.coloros.smartdrive
```

删除屏幕共享
```sh
adb shell pm uninstall -k --user 0 com.coloros.sharescreen
```

删除应用时间使用
```sh
adb shell pm uninstall -k --user 0 com.coloros.digitalwellbeing
```

删除扫一扫服务后台
```sh
adb shell pm uninstall -k --user 0 com.coloros.colordirectservice
```

删除车联
```sh
adb shell pm uninstall -k --user 0 com.oplus.ocar
```

删除数据服务平台
```sh
adb shell pm uninstall -k --user 0 com.coloros.sceneservice
```

删除健康数据平台
```sh
adb shell pm uninstall -k --user 0 com.oplus.healthservice
```

删除融合搜索服务
```sh
adb shell pm uninstall -k --user 0 com.oplus.dmp
```

删除反馈工具箱
```sh
adb shell pm uninstall -k --user 0 com.oplus.logkit
```

删除智慧能力服务
```sh
adb shell pm uninstall -k --user 0 com.oplus.deepthinker
```

删除支付保护
```sh
adb shell pm uninstall -k --user 0 com.coloros.securepay
```

删除omoji
```sh
adb shell pm uninstall -k --user 0 com.oplus.omoji
```

删除metis
```sh
adb shell pm uninstall -k --user 0 com.oplus.metis
```

删除多端设备协同onet
```sh
adb shell pm uninstall -k --user 0 com.oplus.onet
```

删除第三方云端解决方案thirdkit
```sh
adb shell pm uninstall -k --user 0 com.oplus.thirdkit
```

删除crashbox
```sh
adb shell pm uninstall -k --user 0 com.oplus.crashbox
```

删除shelper
```sh
adb shell pm uninstall -k --user 0 com.daemon.shelper
```

删除主题商店（需要在，恢复出厂或者新开始不登陆账号的情况下，才可操作）(影响：无法下载第三方主题壁纸字体。自带的不受影响)
```sh
adb shell pm uninstall -k --user 0 com.heytap.themestore
```

建议停用
```sh
adb shell pm disable-user com.heytap.themestore
```

参考：
 - [精简oppo系统 coloros13 adb停用删除自带app，卸载快应用](https://bbs.oneplus.com/thread/6003405?mod=viewthread&tid=6003405)
 - [OPPO系统内置软件卸载指南](https://blog.sunflyer.cn/archives/689)
