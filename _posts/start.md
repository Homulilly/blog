---
title: 'Hello Typecho :)'
date: 2016-02-24 17:22
tags:
---

从很久以前就接触了Wordpress，然而到现在，还是什么文章都没有留下来。<(=╯▽╰=)>  
手里虽然一直都有着VPS，但最近来说也都是拿来做代理用了，其余的什么都没有做，把博客扔到了Blogger上，不过还是一个样子，并没有说去写过什么像样子的文章。

前几天，发现了Typecho这个东西，感觉上很清爽简洁，便想尝试一下。  
然后，自己在DTI的OpenVZ的VPS也被干扰的几乎不能使用了，便打算一起换掉。
<!--more-->
于是这次顺带搭上了软娘Azure的东风，又重新折腾起来了。  

系统还是顺手的选择了Ubuntu 14.04,虽然16.04马上就出来了，不过感觉区别也不是很大。  
装好PHP、MySql、Apache，下载TypeCho的安装包，一路倒还是蛮顺畅的。

然后，主题选择了 [Jrotty][1] 制作的这份主题，我觉得点击 头像/搜索 出现全屏About的界面这个功能不是那么好使，感觉不太优雅， <s>还有这停不住的 “发呆”</s>  。不过整体的风格还是比较喜欢的。  

0.0

于是大体框架，装起来还是蛮顺手，不是太费事，不过真的一些小细节说起来，还是全部都得靠G娘才能搞定。  

整体都弄完了后，现在的就还有些问题需要处理了：  
1.网站文件目录的权限问题，现在不太清楚要设置成什么的样子的；   
2.就是这个主题的评论的问题了，说实话第三方不是太那么的想用，这主题看样子自己不带。。。再说吧，反正也没人； `Done`
3.首页侧栏的这些链接都还没有页面(´・ω・｀) `Done`

#记录   
* Let's Encrypt it! ( [Apache && Ubuntu 14.04][2] )  
    ```  
    #准备 & 将letsencrypt放到/opt/letsencrypt
    sudo apt-get update && sudo apt-get upgrade
    sudo apt-get install git

    sudo git clone https://github.com/letsencrypt/letsencrypt /opt/letsencrypt  

    #签发证书
    cd /opt/letsencrypt
    ./letsencrypt-auto --apache -d example.com -d www.example.com

    #证书续期
    ./letsencrypt-auto certonly --apache --renew-by-default -d example.com -d www.example.com
    ```
* Force https && without www  
    ```
    #add this to .htaccess file
    RewriteCond %{HTTP_HOST} ^(www\.)(.+) [OR]
    RewriteCond %{HTTPS} off
    RewriteCond %{HTTP_HOST} ^(www\.)?(.+)
    RewriteRule ^ https://%2%{REQUEST_URI} [R=301,L]
    ```

#主题相关的修改   
* 评论以及评论的样式  
  add style to head.php  
  来自Typecho主题 [NexT.Pisces][3]   
  Using Disqus - 2016.02.26  
* <s>Markdown相关的修改 （想要删除线，但还没有动手改，考虑中</s>  
  <s>支援 GitHub 扩展 Markdown | var [梦姬博客][4]</s>   
  <s>PS：额，这Markdown段落换行要有空行，不带空行的要加 `<br />`，而Typecho自带的加 `<br />` 会有空行，坑，删除线，只好先靠脑补了。</s>  

* Disable  ( ゜- ゜)つ & Lang 简<->繁体   

* 侧栏  
  Weibo -> Google+ : `fa-weibo -> fa-google-plus-square @sidebar.php`
  @ -> Link : `fa-at -> fa-link @sidebar.php`
* Modify #about ->   
  Remove Search & Time   
  add `background-attachment: fixed` to footer.php  
  change #about `overflow：hidden` -> `overflow-y:auto` in css  
* Remove Copy Protect : `Done`  

  [Typecho Markdown Syntx](https://sourceforge.net/p/typecho/discussion/markdown_syntax)  

  | TABLE | ATEST |   
  | ----- | ----- |
  | AAAAA | BBBBB |
  | CCCCC | DDDDD |     Failed @2016.02.28



  [1]: http://qqdie.com/
  [2]: https://www.digitalocean.com/community/tutorials/how-to-secure-apache-with-let-s-encrypt-on-ubuntu-14-04
  [3]: https://github.com/newraina/typecho-theme-NexTPisces
  [4]: http://blog.jixun.org/post/40.html
