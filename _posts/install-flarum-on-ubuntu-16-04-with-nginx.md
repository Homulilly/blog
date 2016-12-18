---
title: Ubuntu 16.04 &amp; Nginx 下配置 Flarum
tags:
  - Ubuntu
  - Nginx
  - Let's Encrypt
categories:
  - Linux
date: 2016-06-06 15:12:00
---

感觉 Flarum 论坛的界面很简洁，比较讨人喜欢。打算搭一个尝试一下 <s>虽然只是我一个人用233333</s> ，以前一直都是使用的 Apache ，这次也换了 Nginx 。

## 准备

参考链接：  
*   Flarum Offical Site : [Flarum Installation ](http://flarum.org/docs/installation/)
*   Nginx 安装配置 : [Linux下Nginx、MySQL、PHP5、phpMyAdmin安装与配置](http://www.pythoner.com/197.html)

系统环境： Ubuntu 16.04 LTS (x64)
<!--more-->
## 安装基础环境  
### 1. 安装 Nginx

    sudo apt-get install nginx

配置文件在/etc/nginx下  
程序文件在/usr/sbin/nginx下  
日志文件在/var/log/nginx下  
网站目录在 /var/wwwhtml/  

### 2. 安装 PHP

    sudo apt-get install php-cli php-cgi php-fpm php-mcrypt php-mysql php-gd php-common php-zip php-curl

php-fpm 是 Nginx 配合 PHP 使用的模块。  
php-zip 、 php-curl 在安装 Flarum 时会用到。

在 Nginx 的站点配置中 location ~* .php$ { } 字段中加入  

    fastcgi_param  SCRIPT_FILENAME  /var/www/html/$fastcgi_script_name;
    include        fastcgi_params;
    fastcgi_index  index.php;

`sudo service nginx reload`

### 3. 安装 MySQL

    sudo apt-get install mysql-server mysql-client

### 4. 安装 phpMyAdmin

下载链接: [https://www.phpmyadmin.net/downloads/](https://www.phpmyadmin.net/downloads/)

## 安装 Flarum

#### 首先安装 Composer ：   
Link: [https://getcomposer.org/download/](https://getcomposer.org/download/)

    php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
    php -r "if (hash_file('SHA384', 'composer-setup.php') === '070854512ef404f16bac87071a6db9fd9721da1684cd4589b1196c3faf71b9a2682e2311b36a5079825e155ac7ce150d') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
    php composer-setup.php
    php -r "unlink('composer-setup.php');"

#### 安装 Flarum

    composer create-project flarum/flarum . --stability=beta

Nginx 站点配置文件中加入下面内容
```
    location / { try_files $uri $uri/ /index.php?$query_string; }
    location /api { try_files $uri $uri/ /api.php?$query_string; }
    location /admin { try_files $uri $uri/ /admin.php?$query_string; }

    location /flarum {
        deny all;
        return 404;
    }

    location ~* \.php$ {
        fastcgi_split_path_info ^(.+.php)(/.+)$;
        #change php5-fpm.sock ->  php7.0-fpm.sock;
        fastcgi_pass unix:/var/run/php7.0-fpm.sock;   
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_index index.php;
    }

    location ~* \.html$ {
        expires -1;
    }

    location ~* \.(css|js|gif|jpe?g|png)$ {
        expires 1M;
        add_header Pragma public;
        add_header Cache-Control "public, must-revalidate, proxy-revalidate";
    }

    gzip on;
    gzip_http_version 1.1;
    gzip_vary on;
    gzip_comp_level 6;
    gzip_proxied any;
    gzip_types application/atom+xml
               application/vnd.ms-fontobject
               application/x-font-ttf
               application/x-web-app-manifest+json
               application/xhtml+xml
               application/xml
               font/opentype
               image/svg+xml
               image/x-icon
               text/css
               text/plain
               text/xml;
    gzip_buffers 16 8k;
    gzip_disable "MSIE [1-6]\.(?!.*SV1)";
```
然后访问域名，进行安装。  
<s>记得我安装时这一步不能正常进行，F12看了一下，各种资源404。通过编辑 config.php 文件，把url 后面 domian.com/index.php 的index.php 去掉后可以了</s>

#### 安装中文语言包

Github 页面 ： [https://github.com/Flarum-Chinese/flarum-ext-simplified-chinese](https://github.com/Flarum-Chinese/flarum-ext-simplified-chinese)

`composer require jsthon/flarum-ext-simplified-chinese`

结果提示killed composer require jsthon/flarum-ext-simplified-chinese  
没有成功安装。

执行 `composer install` 提示下面内容  
> Warning: The lock file is not up [How To Secure Nginx with Let's Encrypt on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-16-04)
>to date with the latest changes in composer.json. You may be getting outdated dependencies. Run update to update them.  

然后执行 `composer update` 继续提示下面内容

> The following exception is caused by a lack of memory and not having swap configured
> Check [https://getcomposer.org/doc/articles/troubleshooting.md#proc-open-fork-failed-errors](https://getcomposer.org/doc/articles/troubleshooting.md#proc-open-fork-failed-errors) for details  

按照链接中的指导开启 swap

    /bin/dd if=/dev/zero of=/var/swap.1 bs=1M count=1024
    /sbin/mkswap /var/swap.1
    /sbin/swapon /var/swap.1


再执行 `composer install`  就好了。

#### 配置 Flarum 邮件  
直接在数据库 setting 里添加以下字段   
```
mail_driver: smtp
mail_host: ...
mail_from: ...
mail_port: ...
mail_username: ...
mail_password: ...
mail_encryption: ...  
```

#### 配置 Let's Encrypt 的证书  

参考 DO 上面的教程 ： [How To Secure Nginx with Let's Encrypt on Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-ubuntu-16-04)

结果一直报错

> Failed authorization procedure. bbs.illya.pw (http-01): urn:acme:error:connection :: The server could not connect to the client to verify the domain :: DNS problem: query timed out looking up CAA for domain.com
>
> IMPORTANT NOTES:
>
>   The following errors were reported by the server:  
>   Domain: domain.com  
>   Type:   connection  
>   Detail: DNS problem: query timed out looking up CAA for domain.com

搜索了一大圈都没找到有效的解决办法，最后试着在 Nginx 的站点配置文件中打开了 443 端口后可以了，我也不知道是突然 DNS 的问题解决了，还说打开 443 端口就好了。看 DO 上面的那篇教程文章上开启端口明明也是后面的步骤。
