---
title: 解决 macOS 终端连接 SSH 闲置几分钟后卡住的问题
date: 2025-06-12 16:57:21
tags:
 - macOS
categories:
 - Note
---

我用 macOS 默认终端使用 SSH 连接服务器后总是闲置几分钟后就会卡住，无法操作，需要关闭终端重新打开。由于 macOS 的 SIP 对系统文件的保护，在 `/etc/ssh/ssh_config` 中加入心跳消息配置没有生效。 

可以在 `～/.ssh/config` 文件开头加入下面的内容：
```
Host *
    ServerAliveInterval 30
    ServerAliveCountMax 10
```

表示每 30s 向服务器发送一次心跳连接，10 次失败后断开。
