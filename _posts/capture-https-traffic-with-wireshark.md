---
title: 使用 Wireshark 抓取 HTTPS 数据包
date: 2017-01-16 18:26:09
tags:
 - Wireshark
categories:
 - Note
---
偶尔需要抓包一下获取的点信息， HTTP 好弄，然而 HTTPS 的流量是加密的，这时 Wireshark 看不到想要的信息。郁闷，搜索一下，如果只是抓本机的浏览器的包的话，稍微设置一下就可以了，转来记录一下。

原理就是设置 `SSLKEYLOGFIL` 这个环境变量，这样 Chrome/Firefox 会存储 HTTPS 会话的密钥到该环境变量所指定的文件中。然后在 Wireshark 中填入该文件的路径就可以了。

<!--more-->
Ubuntu 设置环境变量并启动浏览器：
```
export SSLKEYLOGFIL=~/sslkey.log

google-chrome
```

访问 HTTPS 网站，查看 `~/sslkey.log` 文件内是否出现内容。然后在 Wireshark 中填入该文件的路径。  

Wireshark 设置： 编辑 > 首选项 > Protocols > SSL > `(Pre)-Master-Secret log filename`  

这时用 Wireshark 抓包就可以看到 HTTPS 会话的内容了。在顶部过滤框中输入 `http` 即可。  

Wireshark Display filter 语法： [Building display filter expressions](https://www.wireshark.org/docs/wsug_html_chunked/ChWorkBuildDisplayFilterSection.html)

不过这个只对本机有效，对于其他设备的 HTTPS 流量没用，不过可以使用 [Charles](https://www.charlesproxy.com/) 使用中间人攻击的方式来进行分析，不过我现在是用不到，记一下。

参考链接：  
[Wireshark抓取HTTP压缩包和SSL包](https://my.oschina.net/swingcoder/blog/533979)
[Charles是如何抓取https数据包的](http://legendtkl.com/2015/11/30/charles-https/)
