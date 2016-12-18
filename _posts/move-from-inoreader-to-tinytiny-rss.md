---
title: 准备从 Inoreader 更换到 Tiny Tiny RSS
date: 2016-03-25 22:22:00
tags:
  - TinyTiny Rss
---

说起来，虽然自己感觉自己并不算是RSS重度用户，但是一开始从Google Reader开始，也是养成了使用RSS服务的习惯，把众多网站的信息集中到一起，然后在挑选自己感觉兴趣的阅读还是比较方便的，而且在记录方便也有一定的优势。  
然而后来 Google 关闭了 [Google Reader][1] ， 真是一件让人遗憾的事情。  
<!--more-->
随后就一直在找相应的替代品。  
一开始用的是Feedly，去搜了一下自己的Post， 使用时间是从 2013年5月 到 2014年6月，也就一年多一些，看了下中间的吐槽，没有满意的地方，除了界面看起来不错的样子，尤其是后来Feedly被GFW认证了，使用起来更麻烦，记得当时免费版还不能搜索，于是寻找替代品的行动并没有停下来。  

Feedly之后，使用的是Inoreader，并且一直使用至今。  
总体来说的感觉还是很不错的，首先是界面很简洁，尤其是后来发现可以订阅Twitter的信息流后更加的喜欢了起来。  
不过感觉有些慢， <s>且有广告 (嘛，这个毕竟免费版，其实还是可以理解的)</s>

前几天，看到了别人发的 Tiny Tiny RSS 的 post，去了其 [官网][2] 一看 ，感觉官网的界面很不错，便以为其界面也会很棒，以前在寻找 Google Reader 替代品的时候，其实有了解过 Tiny Tiny RSS ，但并没有尝试搭建过，因为觉得界面会比较丑，就是跟不上时代的感觉。  
这次还是好奇的尝试搭建了一下，然后果然界面还是比较丑的，然后又分别尝试了 Feedly / GReader / Reeder 这三个主题，只有 Feedly 的还能看一些，Reeder是比较丑，GReader的那个则是没有宽版。  
然后快速扫描了一眼插件与功能，只有一个替换Tumblr的图片为大图版本是比较感兴趣的。  
而且默认用户名为admin，且不能修改。  
在处理更新feed上也出现了问题。  

于是，稍微体验一下，还是觉得继续留在 Inoreader比较好，毕竟比较省事一些，而且搜索了一大堆没有找到现在 Tiny Tiny RSS订阅Twitter的方法。  

不过呢，昨天的时候，一打开 Inoreader ，右侧的一个大大的侧栏广告把我惊着了，太影响体验了，我个人是非常的反感这种无所谓的压缩有效可视面积的行为的，包括之前SB Luke的行为。于是便下定决心迁移到 Tiny Tiny RSS 了。虽然现在的Feedly主题看起来说略微的不爽，但是也可以，凑合够用。  

中间过程碰到了一些问题：  
* J/K 快捷键并不是上下导航  
这个发现自带插件中有一个可以把快捷键修改为Google Reader的快捷键，于是解决。  

* 默认用户名为 admin   
自己的服务器搭建的，哪有什么不能改的，去数据库就改掉了。  

* 替换 Tumblr 图片为大图版本的插件部工作  
发现在插件那里光选中是没有用的，要点一次最下的 “使以上选中插件生效” 的选项，好坑，不过解决。  

* 自动更新 Feeds  
一开始，是参照官方的 [指导][3] 设置 crontab   
```
*/30 * * * * /usr/bin/php /path/to/tt-rss/update.php --feeds --quiet
```
然而不知道为什么，后台进程还在，但是工作的不正常，每次只有零星的源有更新，可以说和没更新一个样。使用的也是对 update.php 有执行权限的用户执行的，不工作，但是不知道为什么。  
后来，请教了别人后，把命令修改成了下面的  
```
 screen -S ttrss /usr/bin/php /path/to/tt-rss/update_daemon2.php
```
可以正常更新feeds了，虽然看了一下更新的频率是一分钟一次，感觉有些高，但是总比不更新的强，时间间隔，等再了解一下再修改吧。  

* 订阅 Twitter   
<s>暂时无解</s>  
Update @2016.04.02 原来有现成的hhh，也可以放在自己的PHP的服务器上。
> Twitter2rss proxy : <http://twitter2rss.nomadic.name/>
> Github : <https://github.com/n0madic/twitter2rss>


* 阅读时，自动标记为已读有1-2s的延时  
不爽，暂时不知道怎么解决

安卓客户端可以通过 News+ 配合其 Tiny Tiny RSS 插件使用。  
于是，使用起来没有太大的问题了，所以正式准备迁移到 Tiny Tiny RSS 了。  

Bye Inoreader.  

---

说起来一开始打算尝试 Tiny Tiny RSS 还是因为这篇文章 [《Tiny Tiny RSS – RSS 阅读解决方案》][4]  ，打开后对其中的截图十份的感兴趣，的确有当年 Google Reader 的感觉，然而实际安装上去发现这个主题并不是宽屏的，所以会显得很拥挤，感觉并不好。  


  [1]: https://www.google.com/reader/about/
  [2]: https://tt-rss.org/gitlab/fox/tt-rss/wikis/home
  [3]: https://tt-rss.org/gitlab/fox/tt-rss/wikis/UpdatingFeeds
  [4]: http://www.appinn.com/tiny-tiny-rss/
