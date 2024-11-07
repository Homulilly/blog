---
title: Web 版 VS Code - Code Server
date: 2024-04-04 13:59:46
tags:
 - Code Server
 - VSCode
categories:
 - Note
---

本来想搜索一下 Andorid 版本的 VS Code ，没有官方的，但是有基于网页版 [Code Server](https://github.com/coder/code-server) 制作的版本。  

于是我也自己部署尝试了一下，使用起来确实没有问题，安装也是非常的简单。

<!--more-->

### 在 Debian 12 中安装
[Install - Debian, Ubuntu](https://coder.com/docs/code-server/latest/install#debian-ubuntu)  
手动安装的的话，访问 [Github - Releases](https://github.com/coder/code-server/releases) 下载 dpkg 文件就可以了  

```sh
curl -fOL https://github.com/coder/code-server/releases/download/v$VERSION/code-server_${VERSION}_amd64.deb

sudo dpkg -i code-server_${VERSION}_amd64.deb

sudo systemctl enable --now code-server@$USER

// 关闭与启动
systemctl start code-server@$USER
systemctl stop code-server@$USER
```

运行后会在 `~/.config/code-server/config.yaml` 生成配置文件，初次生成包含默认的 `127.0.0.1:8080` 与访问密码，如果是本地使用，直接访问 `http://127.0.0.1:8080` 即可。  

如果是部署在服务器中，建议使用 nginx 进行反代然后就行访问。  
Code Server 有设计安全访问限制，限制每分钟两次、每小时十二次的密码访问尝试。    
- [Expose code-server](https://coder.com/docs/code-server/latest/guide#expose-code-server)
- [FAQ](https://coder.com/docs/code-server/latest/FAQ)

### 扩展

看了一下扩展并不是从官方 VS Code 商店而是从 [https://open-vsx.org/](https://open-vsx.org/) 安装的，可以正常安装中文语言包。  
图标主题也可以正常使用。  

也可以从 [官方商店](https://marketplace.visualstudio.com/) 扩展主页，点击 `Download Extension` 下载 vsix 文件  
然后手动安装  
```sh
code-server --install-extension chadalen.vscode-jetbrains-icon-theme-2.18.0.vsix
```

在服务器上使用，切换文件夹没有原版方便，推荐安装扩展 Project Manager：
 - [Project Manager](https://marketplace.visualstudio.com/items?itemName=alefragnani.project-manager)
 - [Open VSX Store](https://open-vsx.org/extension/alefragnani/project-manager)

### 设置
#### 字体
和普通版本一样，按打开设置(`Ctrl + ,`)，搜索字体，然后修改即可。 
```
'Source Code Pro', '微软雅黑', monospace
```

#### 终端
`` Ctrl + ` `` 打开终端  
`` Ctrl + `Shift` + ` `` 新建一个终端

终端右上角的菜单可以选择默认终端配置。  

![code-server](https://m.nep.me/blog/post/code-server.png)

### 在 Android 上使用 
直接在 Chrome 中使用有地址栏看起来很是不爽，不过只要点击 Chrome 菜单，选择 `添加到屏幕` 就可以全屏使用了。  
效果也是非常的不错。  
只是长按屏幕是调出邮件菜单，无法范围选择文本，还好我的 Pad 选配了键盘。

### Termux 安装
甚至也可以直接部署在 Termux 中：[Termux - Install](https://coder.com/docs/code-server/latest/termux#install)  

看起来也是非常简单：
1. 安装 Termux
2. 运行 `pkg install tur-repo`
3. 运行 `pkg install code-server`
4. 运行 `code-server` 启动

不过我暂时还没有需求，所以还未尝试。 

### 修改 LOGO
默认的 LOGO 不是很好看，可以自己替换，如果是使用 deb 安装的话，直接替换 `/usr/lib/code-server/src/browser/media/` 目录下的图标文件即可。  
一共有五个：
```sh
favicon-dark-support.svg
favicon.ico
favicon.svg
pwa-icon-192.png
pwa-icon-512.png
pwa-icon.png
```
可以在 [Yesicon](https://yesicon.app/) 方便的下载 svg 与 png 的格式的图片。  

--- 

这个文章就是在 Code Server 中创建、完成、推送的。
可以在不同设备中无缝切换使用，编辑文本时不同窗口也会自动同步。   
所以说有一个 All in ONE 主机还是蛮不错的。又给我的 vivo Pad 添加了一点生产力。    