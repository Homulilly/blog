---
title: ConEmu with Bash on Windows
date: 2016-09-19 07:21:00
tags:
 - ConEmu
 - Bash on Windows
categories:
 - Note
---
本子是 256G 的 SSD ，之前尝试了双系统，无奈双 128G 实在是太憋屈，在微软出了 Bash on Windows 后权衡了一下，还是把本子上的 Ubuntu 拆掉了，毕竟不常用本子，偶尔还是有需求拿来跑一下 Win 的程序。  

Bash on Windows 说起来感觉还是不错，就是一点 cmd 太难用，长得又丑，新版是加了字体，但是配置无法保存什么鬼。
XShell 不知道是不是我姿势不对，无法运行 bash 。所以还是下载了 ConEmu, 虽然还是没有右键菜单，不过看起来舒服多了。  
<!--more-->

<s>然而 oh-my-zsh 才是真爱哇。</s>  

运行 bash.exe 后自动切换为 zsh ，可以将下面内容加入 .bashrc :
```
# Launch Zsh
if [ -t 1 ]; then
exec zsh
fi
```

以及直接在 ConEmu 里运行了 bash 的话，是无法使用方向键的，真是坑，不过看了下自带的 task 中的命令是 `%windir\system32\bash.exe -cur_console:p` ，总之加上了 `-cur_console:p` 就可以正常使用方向键了，再加一个 `~` 即可运行后自动进入 $HOME 路径。

加上之前的自动切换为 zsh ，真是 Prefect (ﾉ>ω<)ﾉ ，虽然右键还是感觉略残念就是。
