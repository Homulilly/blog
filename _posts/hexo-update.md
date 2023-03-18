---
title: 更新 Hexo 和 NEXT 主题
date: 2023-03-18 23:30:05
tags: 
 - Hexo
 - Twikoo
categories:
 - Talk
---
博客换到 Hexo 之后，就一直没有更新过 Hexo 和 Next 主题的版本，现在版本差的有些多。    
```sh
hexo --version
hexo: 3.4.1

# Now
hexo --version
INFO  Validating config
hexo: 5.4.2
```
更新一下 
<!--more--> 

## 1. 设置环境

```sh
# 安装 Hexo
npm install hexo-cli -g
hexo init hexo

cd hexo
npm install

# 安装主题
# 注意仓库是否是最新的 8.x 的版本
git clone https://github.com/next-theme/hexo-theme-next themes/next

# 拉取 blog 的备份
rm -rf source
git clone https://github.com/Homulilly/blog.git source
```

## 2. 编辑配置
修改配置 `_config.yml` 和 `theme/next/_config.yml`   
- 语言由 `zh-hans` 替换为 `zh-CN`

## 3. 问题处理
预览
```sh
hexo g && hexo s
```
与在线版本对比后，进行修改

- **关闭载入动画**  
编辑 `theme/next/_config.yml` 
```yml
motion:
  enable: false
```
- **站点 `Title` 字体变动**  
设置为 `Lato` 
- **字体变大**  
字体 `size` 设置为 0.9 后与之前的一致
```yml
  global:
    external: true
    family: Ubuntu
    size: 0.9
```
- **图片** 
  - **启用点击**  
  配置文件启用 `fancybox` 
  - ~~图片居中~~
  - ~~给图片加个边框~~    
  - **移除 Fancybox 的 Image Caption** 
  将下面的内容加到文件 `next/source/css/_schemes/Mist/_posts-expand.styl`中，加到 `.post-tags` 与 `.post-nav` 中间
  ```css
  /*一通操作猛如虎，打开暗黑模式丑上天  
  .post-body img {
    margin: 0 auto;
    padding: 3px
    border: 1px solid #ddd;
  }*/
  .post-body .image-caption {
    display: none;
  }
  ```
- 顶部导航的 MENU 改为靠右  
编辑文件 `themes/next/source/css/_schemes/Mist/_menu.styl`  
```css
.menu {
  margin: 0;
  // 添加
  text-align: right;
...
}
```

- **文章多了 `更新于...`**  
编辑 `theme/next/_config.yml`
```yml
post_meta:
  updated_at:
    enable: false
```
- **Footer 红心颜色**  
设置为 `"#666"`
- **About 页面， HTML + Markdown 嵌套后不解析**
就一行，直接用 HTML 写了
- **启用搜索**
```yml
npm install hexo-generator-searchdb --save

#_config.yml
search:
  path: search.xml
  field: post
  format: html
  limit: 10000

# /theme/next/_config.yml
# Local search
  local_search:
  enable: true
```
- **Disqus load after click**
不会改了，直接换 Twikoo   

## 4. 设置 Twikoo

### 使用 docker 安装
```sh
docker pull imaegoo/twikoo

docker run --name twikoo -p 127.0.0.1:8080:8080 -v [path]/data:/app/data -d imaegoo/twikoo
```

### 设置 Nginx 反代
```bash
upstream twi {
    server 127.0.0.1:8080;
}

server{
  //...
  location / {
    proxy_pass http://twi;
  }
}    
```
### 设置自启动  
```sh
docker update --restart=always  twikoo 
```

### 测试
访问使用的域名，如果成功则提示

```json
{"code":100,"message":"Twikoo 云函数运行正常，...","version":"1.x.x"}
```

Ref: 
- [Twikoo小白私有化部署教程,迁移腾讯云](https://xiaoniuhululu.com/2022-08-09_twikoo_privatization_deployment_tutorial/)

### 部署到 Hexo 

```sh
#NexT version 8.15.0 

npm install hexo-next-twikoo@1.0.3
```

编辑配置文件，添加下面内容
```yml
twikoo:
  enable: true
  visitor: false
  envId: {url}
```

```sh
hexo g && hexo s
```

预览，然后点击 ⚙ 设置管理密码，然后进入面板设置。

## 5. 推送更新

```sh
npm install hexo-deployer-git

hexo d
```
