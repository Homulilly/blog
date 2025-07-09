---
title: 使用 VMware Fusion 在 macOS 中创建 Windows ARM 虚拟机
date: 2024-12-12 15:27:29
tags:
 - macOS
 - Windows ARM
 - VMware Fusion
categories: Note
---

之前 VMware Workstation 和 Fusion 彻底免费了，外加也可以下载 Windows on ARM 的镜像了。  

安装下来体验一下。  

与使用 UTM 对比
 - UTM 可以方便的设置共享文件夹，从虚拟机访问宿主机的文件夹
 - UTM 安装 Window11 时会自动重启到设置本地账户
 - Fusion 使用感觉更流畅一些，UTM 偶尔会卡顿一下

<!--more-->

## 免登录下载 VMware Fusion 

可以直接使用 Homebrew 安装
```
brew install vmware-fusion
```

直接去官网下载 VMware Fusion 需要登陆，不打算注册相关账户，于是搜索了一下，发现了一个免登录下载的方法。  

就是从更新接口下载 https://softwareupdate.vmware.com/cds/vmw-desktop/

不过下载后需要手动解压才可以安装。  

依次点击 Fusion > 13.6.1 > 24319021 > universal > core > com.vmware.fusion.zip.tar 进行下载

下载后需要解压两次得到 `VMware Fusion.app` 复制到 应用程序(Application) 中。 

然后打开 VMware Fusion ，按照提示进行授权就可以了。

## 下载 Windows 11 ARM64

访问微软官方网站下载 
https://www.microsoft.com/zh-cn/software-download/windows11arm64

## 安装 Windows 

打开 VM Fusion ，把下载的镜像拖放到窗口中，然后按照步骤进行

- 固件选择 UEFI
- 选择加密，需要生成一个 TPM 密码，可以自动生成，要保存好
- 进入完成界面，默认说 4G 内存，2 核心CPU，点击自定义设置可以修改虚拟机名称并修改配置

macOS Fusion 鼠标回到宿主机的快捷键是 `Control + Commond`

### 安装过程
启动后显示 黑屏 + `Press any key boot from CD/CDROM` 时要马上按一个键，不然很快就跳过这个步骤了（可以等超时后进入蓝色的 BIOS 界面再选择 **Boot normally**）。

然后就进入 Windows 的安装界面了 

![](https://m.nep.me/blog/post/use-fusion-install-win-arm-step.png)

按照提示进行点击下一步
 - 选项安装选项：**安装 Windows 11**
    - **勾选** 同意 I agree everything will be deleted including files, apps, and settings
 - 产品密钥界面选择左下角 **我没有产品密钥**
 - 映像选择 **Windows 11 专业版**
 - **接受** 许可协议
 - 选择磁盘直接点击 **下一步**
 - 最后选择 **安装**

安装过程 Windows 会自动操作，重启几次后进入 Windows 初始设置界面。  

按照步骤选择语言、键盘布局后，在设置网络这一步会卡住 
![Windows 11 Install No Network](https://m.nep.me/blog/post/use-fusion-install-win-arm-no-network.png)

需要按 `Shift + F10` 打开命令窗口，输入下面的命令
```
OOBE\BypassNRO
```

然后会重启，需要重新选择地区、键盘布局，到添加网络时可以选择 **我没有 Internet 连接** 了。 

然后就可以继续，设置账户名称、密码（可以跳过）、隐私选项。

### 安装 VMware Tools
- 点击顶部菜单的 **虚拟机** > **安装 VMware Tools** 。
- 虚拟机窗口弹窗选择安装，但是并不自动安装，只会挂载光驱
- 打开 Windows 的资源管理器，在光驱中手动点击 **Setup** 安装

### 其他设置
我安装后 Fusion 已经自动设置了 Retina 支持，可以在虚拟机 **设置** > **显示** 中查看

安装 VMware Tools 后 Windows 的显示缩放比例是 250%，不是很合适。  

打开 Windows 的设置，显示设置，缩放部分点击 `关闭自定义缩放`，注销登录后，手动设置一个适合的分辨率。

### 删除虚拟机 
右键 Dock 上的 VMware Fusion 图标，选择 **虚拟机资源库**