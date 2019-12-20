---
title: 使用 Scrcpy 操作 Android 手机
date: 2019-12-14 03:38:29
tags: 
 - Note
 - Android
 - Scrcpy
---

`Another EDEN` 不支持多个设备同时使用一个账号。所以不能愉快的用模拟器划水。  

于是搜了一下有没有可以在电脑上显示并控制 Android 屏幕的软件，发现果然是有的，当然免费的还是最好的。  

`Scrcpy` ([Github](https://github.com/Genymobile/scrcpy)) 可以通过 adb / adb over wifi 来控制屏幕，甚至都不需要在手机上安装软件。  

<!--more-->
### 1. 首先下载 ADB 与 Scrpy

- Scrcpy 目录下包含了 adb.exe ，所以下载 Scrcpy 后可以直接使用  
[Github Releases](https://github.com/Genymobile/scrcpy/releases)

- 单独下载 ADB  
[Android SDK Platform-Tools](https://developer.android.com/studio/releases/platform-tools?hl=zh_cn) 其中包含了 adb.exe   


### 2. 手机上 USB 调试

- 开启开发者选项
- 开启 USD 调试
- 开启网络 ADB 调试

### 3. 使用 USB / WIFI 链接 ADB 调试

#### USB 链接  

1. 将手机与电脑通过 USB 数据线链接，手机上弹出 ADB 调试权限请求时，选择允许。  
2. 切换到 `Scrcpy` 的文件夹，在资源管理器路径框中输入 cmd 即可直接在当前目录下打开命令提示符窗口  
3. 执行 `adb devices` 查看是否已成功链接，如果显示下面内容表示成功连接
    ```
    List of devices attached
    xxxxxxxx        device
    ```
4. 这是可以直接输入 scrcpy , 会弹出屏幕窗口

#### 设置 ADB over WIFI 

emm... 拿来玩游戏，一直插着 USB 不太好，可以设置使用 WIFI 连接 ADB ，不过流畅性不如直接插线就是了。  
就是每次使用 WIFI 链接之前都要先插上 USB 设置一下。  

1. 开启 `wifi-debug-mode`
   ```
   adb tcpip 5555
   ```
   5555 是监听的端口，执行后如果提示下面内容表示设置成功
   ```
   restarting in TCP mode port: 5555
   ```
2. 查看自己手机的 IP 地址
3. 断开 USB 数据线
4. 连接
   ```
   adb connect IP:PORT(5555)
   ```
5. 使用 `adb devices` 查看状态
   ```
   List of devices attached
   192.168.1.x:5555      device
   ```
6. 执行 `scrcpy` 打开窗口
7. 不用了之后可以执行 `adb usb` 切换回 USB 模式 或是 `adb disconnect` 断开连接，

### 4. Scrcpy 的参数与快捷键

可以执行 `scrcpy --help` 查看具体参数。  

我用的参数： 
- `-S` / `-turn-screen-off` : 链接时关闭手机屏幕
- `-r video.mp4` / `--record video.mp4` ： 录制屏幕
- `--window-borderless` : 不显示边框
- `--window-title text` ：设置窗口标题名称为 text
- `--crop width:height:x:y` ：裁剪显示窗口，毕竟游戏不支持宽屏，自带黑边

自用：
```
scrcpy -S --window-title Android --crop 1440:2560:0:280
```

一些快捷键
- `Ctrl` + `x` : 重新调整窗口，去掉黑边
- `Ctrl` + `h` / `b` / `s` / `p` : 相当于点击 Home / Back / 多任务 / 电源键
- `Ctrl` + `o` : 关闭手机的屏幕
- `Ctrl` + `↑` / `↓` ： 调节音量
