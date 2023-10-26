---
title: 在 Debian 12 上安装 NodeJS
date: 2023-10-26 16:30:45
tags:
 - Debian
 - NodeJS
categories:
 - Note
---

打算把 Hexo 的源文件直接放在 PVE 的虚拟机里，记录一下使用 [NodeSource](https://github.com/nodesource/distributions) 在 Debian 12 上安装 NodeJS 的命令。

### 导入 Nodesource 的 GPG key
```
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://deb.nodesource.com/gpgkey/nodesource-repo.gpg.key | sudo gpg --dearmor -o /etc/apt/keyrings/nodesource.gpg
```

### 创建 deb repository
需要将命令中的 `/node_$NODE_MAJOR.x` 替换为对应的版本 比如 `https://deb.nodesource.com/node_20.x nodistro`

```
echo "deb [signed-by=/etc/apt/keyrings/nodesource.gpg] https://deb.nodesource.com/node_$NODE_MAJOR.x nodistro main" | sudo tee /etc/apt/sources.list.d/nodesource.list
```

### 安装
```
sudo apt-get update
sudo apt-get install nodejs -y
```

### 验证
```
nodejs
Welcome to Node.js v20.9.0.
Type ".help" for more information.
> 
```


