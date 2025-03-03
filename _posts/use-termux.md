---
title: Termux 的配置记录
date: 2024-04-03 19:12:10
tags:
 - Termux
 - Android
categories:
 - Note
---

很早之前听说过 Termux ，一直以为只是一个 SSH Client，由于之前买的 Juice SSH 已经数年没有更新了，这次打算换一个使用。  
结果装上才发现原来 Termux 是一个 Linux 环境的终端模拟器，功能十分的强大，相见恨晚。  
不需要 Root 即可使用，支持包管理，可以安装 git, python ... 

记录一下设置过程。
<!--more-->

### 简介与下载 
- [Termux 官网](https://termux.dev/cn/index.html)
- [Termux Wiki](https://wiki.termux.com/wiki/Main_Page)
- [Github - Termux APP](https://github.com/termux/termux-app)
- [F-Droid 下载地址](https://f-droid.org/packages/com.termux/)

Google Play 里面也有，但是已经很久没有更新，使用时会提示版本过低。从 F-Droid 或是 Github 安装即可。  

### 更新仓库 
安装完成后，运行 `termux-change-repo` 即可选择仓库源，然后就会自动更新。我是执行了两次才更新到最新。    
包管理命令是 `pkg` 兼容 `apt` ，也可以使用 `dpkg` 安装 `deb` 文件 。  

### 基本操作
- 缩放文本： 双指放大或是缩小
- 菜单：长按屏幕
- 切换 Session： 侧边栏，与现在安卓的全局手势有点冲突，但是多试试还是能呼出来的
- 快捷键： `音量-` 模拟 `Ctrl`音量加键也可以作为产生特定输入的特殊键.
    - `音量加 + E` -> Esc 键
    - `音量加 + T` -> Tab 键
    - `音量加 + 1` -> F1（音量增加 + 2 → F2… 以此类推）
    - `音量加 + 0` -> F10
    - `音量加 + B` -> Alt + B，使用 readline 时返回一个单词
    - `音量加 + F `-> Alt + F，使用 readline 时转发一个单词
    - `音量加 + X `-> Alt+X
    - `音量加 + W `-> 向上箭头键
    - `音量加 + A `-> 向左箭头键
    - `音量加 + S `-> 向下箭头键
    - `音量加 + D `-> 向右箭头键
    - `音量加 + L` -> | （管道字符）
    - `音量加 + H` -> 〜（波浪号字符）
    - `音量加 + U` -> _ (下划线字符)
    - `音量加 + P` -> 上一页
    - `音量加 + N` -> 下一页
    - `音量加 + .` -> Ctrl + \（SIGQUIT）
    - `音量加 + V` -> 显示音量控制
    - `音量加 + Q` -> 切换显示的功能键视
    - `音量加 + K` -> 切换显示的功能键视

### 目录结构
```bash
echo $HOME
/data/data/com.termux/files/home

echo $PREFIX
/data/data/com.termux/files/usr

echo $TMPPREFIX
/data/data/com.termux/files/usr/tmp/zsh
```

### Termux-ohmyzsh
[Github - Termux-ohmyzsh](https://github.com/Cabbagec/termux-ohmyzsh/)  

安装命令
```sh
sh -c "$(curl -fsSL https://github.com/Cabbagec/termux-ohmyzsh/raw/master/install.sh)"
```

过程中会弹出是否授权访问文件，选择允许。

- `chcolor` 修改配色方案
- `chfont` 修改字体

### 设置目录软链接
可以直接执行 `termux-setup-storage` 初始化，会自动设置一些目录软链接，但并不都是自己需要到。
可以使用 `ln` 手动设置  
```sh
cd ~
ln -s /storage/emulated/0/Documents documents
```

### 安装常用软件
```sh
pkg install -y git vim openssh curl wget tar
```

```sh
pkg install python -y 
```

### 定制常用按键 
编辑文件 `~/.termux/termux.properties` 设置 `extra-keys`  
下面是我的平板的使用的按键
```
extra-keys = [['ESC', '`',  '/', 'HOME', 'END', 'APOSTROPHE', 'QUOTE', 'ESC' ]]
```

### ESC 按键
不知道是 Android 系统将 ESC 是为返回键，还是我的键盘配件的问题，我使用 Termux + Vim 编辑时无法正常使用 ESC ，需要自己定制。  
搜索了一下发现，可以编辑 `~/.termux/termux.properties` 文件，找到下面的部分
```bash
### Send the Escape key.
### 取消下面一行的注释
back-key=escape
```
这样返回(包括全面屏手势)会被视为 ESC ，比较方便。   

### 问题
使用 git 会报错
```sh
fatal: detected dubious ownership in repository at '/storage/emulated/0/Documents/obsidian'
To add an exception for this directory, call:

git config --global --add safe.directory /storage/emulated/0/Documents/obsidian
```
运行了提示的命令就解决了  

### Termux 不显示 extra keys
许久没有用到 extra keys，拿掉物理键盘后才发现没有了，改了半天配置才发现是我忘记自己隐藏掉了
 - 音量UP + Q 可以显示 / 隐藏
 - 从左上角滑动，可以打开菜单，长按 Keyboard 也可以

### 参考
[Termux 高级终端安装使用配置教程](https://www.sqlsec.com/2018/05/termux.html)