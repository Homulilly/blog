---
title: 更新 Hexo 和 NEXT 主题，使用 Twikoo 评论系统 
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
- **暗黑模式鼠标选中背景分不清**  
修改 `next/source/css/_variables/base.styl` , `$selection-bg = $blue-deep;`，可以直接填颜色代码 
```
$selection-bg = #424242;
```

- **Disqus load after click**
不会改了，~~反正也没有什么评论~~，正好也发现了 Twikoo ，换上。  

## 4. 设置 Twikoo

### 使用 docker 安装
```sh
docker pull imaegoo/twikoo

docker run --name twikoo -p 127.0.0.1:8080:8080 -v [path]/data:/app/data -d imaegoo/twikoo
```

### 设置 Nginx 反代
```bash
server{
  //...
  location / {
    proxy_pass 127.0.0.1:8080;

    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header REMOTE-HOST $remote_addr;
    add_header X-Cache $upstream_cache_status;
    # cache
    add_header Cache-Control no-cache;
    expires 12h;
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

Ref: [Twikoo小白私有化部署教程,迁移腾讯云](https://xiaoniuhululu.com/2022-08-09_twikoo_privatization_deployment_tutorial/)

### 部署到 Hexo 

```sh
#NexT version 8.15.0 

npm install hexo-next-twikoo@1.0.3
```

编辑配置主题文件，添加下面内容
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

## 6. 设置 Links 页面样式

### 添加自定义样式  
先修改主题配置文件，添加自定义样式文件，放在 Hexo 文件夹内，而不是主题的文件夹内
```yml
custom_file_path:
  style: source/_data/styles.styl
```

编辑或是创建对应的 `styles.styl`，加入下面的样式
```css
#links {
    margin-top: 5rem;
}

.links-content {
    margin-top: 1rem;
}

.link-navigation::after {
    content: " ";
    display: block;
    clear: both;
}

.card {
    width: 300px;
    font-size: 1rem;
    padding: 10px 20px;
    border-radius: 4px;
    transition-duration: 0.15s;
    margin-bottom: 1rem;
    display: flex;
}

.card:nth-child(odd) {
    float: left;
}

.card:nth-child(even) {
    float: right;
}

/* 鼠标悬浮时效果
.card:hover {
    transform: scale(1.1);
    box-shadow: 0 2px 6px 0 rgba(0, 0, 0, 0.12), 0 0 6px 0 rgba(0, 0, 0, 0.04);
} */

.card a {
    border: none;
}

.card .ava {
    width: 3rem !important;
    height: 3rem !important;
    margin: 0 !important;
    margin-right: 1em !important;
    border-radius: 4px;
    margin-top: 5px !important;
}

.card .card-header {
    font-style: italic;
    overflow: hidden;
    width: 236px;
}

.card .card-header a {
    font-style: normal;
    color: #2bbc8a;
    font-weight: bold;
    text-decoration: none;
}

.card .card-header a:hover {
    color: #d480aa;
    text-decoration: none;
}

.card .card-header .info {
    font-style: normal;
    color: #a3a3a3;
    font-size: 14px;
    min-width: 0;
    text-overflow: ellipsis;
    overflow: hidden;
    white-space: nowrap;
}
```

### 创建 Links 页面

编辑主题配置文件，设置链接
```yml
menu:
  #...
  links: /links || fas fa-link
```

创建页面

```sh
hexo new page links
```

内容如下
````html
---
title: Links
type: links
---

<div class="links-content">
    <div class="link-navigation">
        {% for link in site.data.links %}
        <div class="card"><img class="ava nomediumzoom" src="{{ link.avatar }}" />
            <div class="card-header">
                <div><a href="{{ link.site }}" target="_blank"> {{ link.name }}</a> </div>
                <div class="info">{{ link.info }}</div>
            </div>
        </div>
        {% endfor %}
    </div>
</div>

------

> 友接格式  
```md
- name: Homulilly
  info: Aroes's Blog | \萌脇舞以/
  site: https://homulilly.com/
  avatar: https://homulilly.com/images/avatar.jpg
```
````

`site.data` 就是 `source/_data` 目录，后面的 `links` 就是等会要创建的 `links.yml` 友链文件。

如果向再添加一组不同的链接，可以再建一个 `linkb.yml` => `{% for link in site.data.linkb %}`。

### 添加友链 

在 `source/_data/` 目录下新建 `links.yml` 文件，按照格式填入链接。

### 部署到 Hexo

`links.yml` 有改动时 `hexo g` 时不会更新 links 页面，先 `clean` 一下就可以了。
```sh
hexo clean 
hexo g && hexo s

hexo d
```
Ref: [Hexo + NexT8 添加友链](https://xiaoniuhululu.com/2022-05-09_Hexo+Next8_add_friends_page/)