---
title: Hello Hexo :)
date: 2016-07-31 14:32:26
tags:
  - hexo
---

之前换到这里的时候，用的是 Typecho ，然后用的主题是从 Hexo 移植过来的，然后就知道了 Hexo ，感觉也是很不错。虽然因为主题数量不多，很多的大家都长得差不多。  

然后，觉得Typecho更新比较慢，感觉和没人维护一样，但是去github看看还是在不断更新的，然后就考虑考虑到下次的话换到 Hexo 看看，觉得迁移起来应该比较方便的样子 （怎么老是在迁移 - -），但是懒，然而还在考虑的时候呢，VPS 挂了，启动不了，发了技术支持，人家说可能我的硬盘有问题了，然后需要我新创键一个机器供他们测试用 （Excuse me?）。然后我新建一个后，直接恢复备份了，旧机器挂掉的原因也就不了了之了（我恢复完数据后，告诉他们说检测完后告诉我一下原因然后我把旧的删掉，结果直接告诉我可以现在就删掉了）  

幸好我有每日备份数据库，自己的RSS的内容都没有丢。之前 Typecho 内容也不多， Typecho -> Hexo 迁移文章的话，看来只能通过 rss 的方式，然而 rss 方式格式会有问题，数量也会有限制，还不如直接手动复制了，毕竟 Typecho 也是通过 Markdown 解释文章。  

Hexo 安装起来很是方便，本地安装 Node.JS 环境即可。部署可以通过 git 的方式。  

安装 Hexo ：[Hexo 官方文档 - 中文](https://hexo.io/zh-cn/docs/index.html)  
配置 git ： [使用 Git Hook 自动部署 Hexo 到个人 VPS](http://www.swiftyper.com/2016/04/17/deploy-hexo-with-git-hook/)    
Next 主题文档 ：[NexT 使用文档](http://theme-next.iissnan.com/)

看起来还是很不错的 :)  
不过每个 post 的地址就是 md 文件的文件名，文件夹里排序看起来有点乱，嘛，安装闲着没事的时候也不看。

<!--more-->

---
Modify Theme NexT.Mist :  

Add style to `[HexoPATH]/themes/next/source/css/_custom/custom.styl`  

```css
// Custom styles.

// rss button style color
.feed-link a {
border-radius: 0px;
color: #ababab;
border: 1px solid #ababab;
}
.feed-link a i {
    color: #ababab;
}

// personal link Vertical
.links-of-author a {
    display: block;
    border-bottom: 0px!important;
    width: 75%;
    margin: auto;
}

// remove the dot before link icon
.links-of-author a:before {
    display: none;
}

//hide tag and categories on top
.menu-item-tags {
	display: none!important;
}

.menu-item-categories {
	display: none!important;
}
```

Hexo 渲染自定义 Html页面：
```
---
layout: false
title: "Error 404: Page Not Found"
comment: false
---

<html>
</html>
```

Ubuntu 14.04 通过 ppa 更新 Nginx：  
```
sudo add-apt-repository ppa:nginx/stable
sudo apt-get update
sudo apt-get install nginx
```

Let't Encrypt auto renew every 2 months:   

`sudo crontab -e`

```
# m h  dom mon dow   command
0 0 1 */2 * /opt/letsencrypt/letsencrypt-auto renew >> /var/log/le-renew.log
5 0 1 */2 * /etc/init.d/nginx reload
```
`/etc/init.d/nginx reload` reload Nginx then use the new cert.

Use git & githooks update Hexo:
```bash
# add a user git
sudo adduser git

# add ssh key
sudo su git
cd ~ && mkdir .ssh
vim authorized_keys
exit

# change /bin/bash to /usr/bin/git-shell of user git
sudo vim /etc/passwd

# create a git repo
sudo mkdir /var/repo && cd /var/repo
sudo chown git:git /var/repo
sudo -u git git init --bare hexo.git

# githooks
sudo -u git vim /var/repo/hexo.git/hooks/post-receive
# add bellow to post-receive
#!/bin/sh
git --work-tree=/var/www/hexo --git-dir=/var/repo/hexo.git checkout -f

sudo chmod +x /var/repo/hexo.git/hooks/post-receive
```
