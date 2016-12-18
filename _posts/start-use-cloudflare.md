---
title: 使用 CloudFlare
date: 2016-10-23 22:15:46
tags:
---

话说这个静态类的站点搬家搬起来真是方便，配好系统后重新部署一下就可以了。  

中间比较麻烦的就是给 Tiny Tiny RSS 搬家，不过搬起来也没啥问题，就是搬完后运行 Dropbox 时老是宕机，宕了几次就不耐烦了。感觉就是一个不靠谱，于是这时想到了 CloudFlare，启用 CloudFlare 的过程可以说非常的简单。

注册完 CloudFlare 帐号后，就是添加域名，CloudFlare 会自行扫描 DNS 记录，然后将 DNS 修改为 CloudFlare 给出的两个就可以了。
```
alex.ns.cloudflare.com
zelda.ns.cloudflare.com
```
修改完 DNS 记录后， CloudFlare 那边状态就变成 Active 了，然后慢慢等待 DNS 信息更新就可以了。  

SSL 证书也是 CloudFlare 自动就部署了，完全无需自己手动的操作。

速度啥的，没啥感觉，反正这 ISP 出国是真的很 (눈‸눈) 。
<!--more-->

题外话：  
吐槽一下 Google Domain，我去修改 Google Domain 修改 DNS 时，添加完后保存不了，我在你这儿买的域名，改个 DNS 都出错，尝试单独添加时 alex 那条一下子就添加好了， zelda 就是怎么都添不上，loading 很长一段时间后来个 Failed ，也不显示原因，我都去敲客服了，然而人家客服表示他那边很正常，让我用隐身模式啥的、给我具体操作步骤啥的，然而我就是加不上（  

不过最后突然又加上了，反正不知道为啥 `_(┐「ε:)_`  
以前还对 G娘 有些信仰时开始用的 Google Domain ，现在感觉也就是一般般，还满贵，不过网页操作啥的感觉比 Namecheap 那边要友好一些。

G娘倒是发过一次莫名的优惠，可以免费注册一个 12$ 的域名，用了，然而那么域名都快过期了也没用过 `>_>`
