---
title: 搭配 Oh My Posh 使用 PowerShell
date: 2023-04-04 03:30:46
tags:
 - PowerShell
categories:
 - Note
---

`CMD` 是很快，但是功能偏少，最近看到了 [Oh-my-posh](https://ohmyposh.dev/) 项目，于是决定试试的 PowerShell 

<!--more-->

##  准备

设置 PowerShell 执行策略，允许运行脚本
```
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned
```

- `AllSigned`: 所有脚本都需要签名，包括本地脚本。
- `Bypass`: 不阻止任何操作，并且没有任何警告或提示。
- `Default`: 设置默认策略,默认为 `Restricted` 。
- `RemoteSigned`: 脚本可以运行，需要受信任的发布者对从 Internet 下载的脚本和配置文件的数字签名。
- `Restricted`: 允许单个命令，阻止运行所有脚本文件。

[关于执行策略 - PowerShell](https://learn.microsoft.com/zh-cn/powershell/module/microsoft.powershell.core/about/about_execution_policies?view=powershell-7.3)

## 配置 Oh My Posh

### 安装 
- [官网](https://ohmyposh.dev/)
- [微软商店](https://ohmyposh.dev/)

或是手动安装
```
Set-ExecutionPolicy Bypass -Scope Process -Force; Invoke-Expression ((New-Object System.Net.WebClient).DownloadString('https://ohmyposh.dev/install.ps1'))
```

打开 PowerShell 输入，创建 `oh my posh` 配置文件
```
notepad $profile
```
保存下面内容
```
oh-my-posh init pwsh | Invoke-Expression
```

{% note info %}
Windows 11 新版本的记事本如果提示找不到指定路径，可以直接输入 `$profile` 输出对应的路径，手动创建一下
或是直接在 `文档` 目录下创建一个 `WindowsPowerShell` 文件夹
{% endnote %}

然后重新打开一个 PowerShell 窗口，查看是否生效

### 修改主题
```
Get-PoshThemes
```
会获得所有主题的列表，并展示出来，然后按照提示选择或是编辑一个自己喜欢配置文件

下面是我自己使用的，与我在 Linux 下使用的 `oh-my-zsh` 有相同的样式
```json
{
   "blocks": [
    {
      "alignment": "left",
      "newline": false,
      "segments": [
        {
          "foreground": "red",
          "style": "plain",
          "template": "[",
          "type": "text"
        },
        {
          "foreground": "red",
          "style": "plain",
          "template": " % ",
          "type": "root"
        },
        {
          "style": "plain",
          "template": "<red>{{ .UserName }}</><red>@</><red>{{ .HostName }}]</> ",
          "type": "session"
        },
        {
          "foreground": "white",
          "properties": {
            "style": "full"
          },
          "style": "plain",
          "template": "<darkGray></>{{ .Path }} ",
          "type": "path"
        },
        {
          "style": "plain",
          "template": "<darkGray>on</> <white>git:</><cyan>{{ .HEAD }}</>{{ if .Working.Changed }}<red> x</>{{ end }} ",
          "type": "git",
          "properties": {
            "fetch_status": true
          }
        },
        {
          "foreground": "darkGray",
          "style": "plain",
          "template": "[{{ .CurrentDate | date .Format }}]",
          "type": "time"
        },
        {
          "foreground": "red",
          "style": "plain",
          "template": " C:{{ if gt .Code 0 }}{{ .Code }}{{ end }} ",
          "type": "exit"
        }
      ],
      "type": "prompt"
    },
    {
      "alignment": "left",
      "newline": true,
      "segments": [
        {
          "foreground": "white",
          "style": "plain",
          "template": ">",
          "type": "text"
        }
      ],
      "type": "prompt"
    }
  ],
  "final_space": true,
  "version": 2
}
```
## 问题
{% note danger %}
VSCode 中打开终端提示
oh-my-posh : 无法将“oh-my-posh”项识别为 cmdlet、函数、脚本文件或可运行程序的名称
{% endnote %}
打开配置
```
notepad $profile
```
将开头的 `oh-my-posh` 修改为 `oh-my-posh.exe` 完整路径

## PowerShell Tips

### 使用 `&&`

使用 `;` 即可，例如  
```
hexo g; hexo s
```

### 设置 `alias` 

首先要允许脚本执行。

在配置文件 `note $profile` 中添加

```
function 别名 { 需要替代的命令，可以包含空格 }
```
例如：
```
function hd {hexo clean; hexo generate; hexo deploy}
```

启动 PowerShell 后可以直接运行 `hd` 替代
```
hexo clean; hexo generate; hexo deploy
```
Ref: 
- [为 Windows PowerShell 设置 User Alias （命令别名）](https://blog.vvzero.com/2019/07/22/set-user-alias-for-windows-PowerShell/)
- [About Alias - Microsoft](https://learn.microsoft.com/zh-cn/powershell/module/microsoft.powershell.core/about/about_aliases?view=powershell-7.3)

### PowerShell 中使用 `curl` 无法接受参数

例如 `curl ip.sb -4` 报错
```
Invoke-WebRequest : 找不到接受实际参数“-4”的位置形式参数。
所在位置 行:1 字符: 1
...
```

这是因为 在 PowerShell 中，`curl` 是 `Invoke-WebRequest` 的别名，可以使用下面的命令移除该别名，直接使用 `curl`
```
Remove-Item alias:curl
```
如果需要持久生效，需要将该命令保存到 `$profile` 文件中。

### `.gitignore` 不生效

如果在 Powershell 中使用 `echo` 创建 `.gitignore`，文件的编码是 `UTF-16LE` 不是 `UTF-8`
使用 `UTF-8` 编码重新创建即可
