---
title: 使用 VuePress 建立一个知识笔记站点
date: 2023-03-24 17:17:51
tags: 
 - VuePress
categories:
  - Note
---

VuePress V2 版本还在开发中，尚未推出正式版。  
使用该版本的原因是希望可以使用主题 [vuepress-reco](http://v2.vuepress-reco.recoluan.com/docs/guide/getting-started.html) 的 2.0 版本。

`vuepress-reco` :  
- 界面、样式美观 
- 页面自动生成 `[TOC]`  
- 预览 [NEP NOTE](https://nepnep.me)
<!--more-->
- [本文在 VuePress 上的预览](https://nepnep.me/vuepress.html)

## 安装 VuePress
参考 [VuePress - 手动安装](https://v2.vuepress.vuejs.org/zh/guide/getting-started.html#%E6%89%8B%E5%8A%A8%E5%AE%89%E8%A3%85)

### 初始化
首先创建并进入一个新的文件夹，打开终端，执行下面的命令
```sh
yarn init
```

### 安装 VuePress v2
```sh
yarn add -D vuepress@next
```

### 设置快捷命令

在 `package.json` 中添加一些 `scripts`

`docs` 指的是存放文档的文件夹，可以随意命名
```json
"scripts": {
  "docs:dev": "vuepress dev docs",
  "docs:build": "vuepress build docs"
}
```

即可使用 `yarn docs:dev` 替代 `yarn vuepress dev docs`    

### 创建文档

```sh
mkdir note
echo '# Hello VuePress' > note/README.md
```

{% note danger Powershell创建的文件打开乱码 %}  
如果在 Powershell 中使用 `echo` 创建文本文档，文件的编码是 `UTF-16LE` 不是 `UTF-8`  
重新创建即可
{% endnote %}

### 预览

```sh
yarn docs:dev
```
终端提示 `vite v4.1.4 dev server running at:` 后即可访问 [http://localhost:8080/](http://localhost:8080/) 查看预览 

### 设置站点信息

创建配置文件 `.vuepress/config.ts`，填入下面的内容然后重新预览
```ts
import { defineUserConfig } from 'vuepress'

export default defineUserConfig({
  lang: 'zh-CN',
  title: '网站标题',
  description: "网站描述",
})
```

确认生效后，我们可以设置主题了

## 安装主题
### 手动安装
参考 [vuepress-reco - 快速开始](http://v2.vuepress-reco.recoluan.com/docs/guide/getting-started.html)，选择手动安装的方法

```sh
yarn add vuepress-theme-reco@next
```

修改配置文件 `config.ts` 
```ts
// .vuepress/config.ts

import { defineUserConfig } from 'vuepress'
import { recoTheme } from 'vuepress-theme-reco'

export default defineUserConfig({
  theme: recoTheme({ 
    /* Option */ 
  })
})
``` 

然后重新执行 `yarn docs:dev` 进行预览

### 设置导航栏

导航栏的格式为下面的 `navbar:[ ]` 部分，加入到 `config.ts` 中的 `theme` 部分
```ts
export default defineUserConfig({
  theme: recoTheme({ 
      { text: '首页', link: '/' },
      { text: '笔记', link: '/note/'}
      { text: 'Github', link 'https://github.com'}
   }),
})
```
这是一个简单的导航，效果如图
![Simple Navbar](https://m.nep.me/blog/post/vuepress-nav-simple.png) 

通过嵌套写法，可以实现下拉菜单，比如下面的配置将实现一个下拉菜单
```ts
{
  text: '配置',
  children: [
    { text: '主题配置', link: '/docs/theme/frontmatter' },
    { text: 'Markdown 扩展', link: '/docs/theme/custom-container' },
    { text: '高级',         
    children: [
      { text: '主题配置', link: '/docs/theme/frontmatter' },
      { text: 'Markdown 扩展', link: '/docs/theme/custom-container' },
      { text: '高级', link: '/docs/theme/custom-catalog-title' },
    ], },
  ],
},
```
![Navbar Dropmenu](https://m.nep.me/blog/post/vuepress-nav-drop.png) 

### 设置侧栏

{% note info 有关自动设置 %}
看主题文档说是可以自动设置： [自动设置系列(Series)](http://v2.vuepress-reco.recoluan.com/docs/theme/auto-set-series.html)  
但是我设置后一开始并未生效，研究一下发现，需要在与 `.vuepress` 目录下建立一个 `docs` 文件夹，其下的文档可以自动生成，而且父文件夹也不能是 `docs`。 
{% endnote %}

手动设置同样是编辑配置文件 `config.ts` ，reco 主题中侧边栏使用的是 `series: { }` ，可以放在 `navbar:[ ]` 的下面。
{% note warning %}
不适用于VuePress的默认主题 
{% endnote %}

示例代码
```ts
series: {
  '/note/': [
    {
      // 分组的名称
      text: '基础',
      // children: [] 内部填写文件名 .md 可以省略
      // 获取文章内第一个标题作为导航名称
      children: [ 'linux', 'git' ]
    },
    {
      text: '高级',
      children: [ 'powershell']
    }
  ]
}
```

上面这段代码，将在 `https://your.site/note/` 下面，生成一个如图的侧边栏。 
![VuePress reco Sidebar](https://m.nep.me/blog/post/vuepress-sidebar.png) 

{% note info %}
如果左侧出现文章的名称显示成了文档的路径，一般是因为嵌套子文件夹导致的，填写完整路径即可

以及要使用 `#` 或是 `yaml` 指定标题才会被自动获取

```
---
title: 页面标题
---
```

或是使用 Markdown 格式

```
# 页面标题
```

{% endnote %}

## 添加自定义样式

创建 `.vuepress/styles/index.css` 文件，然后在里面加入样式代码即可。

{% note warning 自用的样式 %}
```css
/* 直角主义 */
code {
    border-radius: 2px !important;
}

.code-group {
    border-radius: 0 !important;
}

div[class*=language-]{
    border-radius: 0 !important;
}

.custom-container {
    border-radius: 0 !important;
}

/* 给图片加个边框 */
p > img {
    margin: 0 auto;
    border: 1px solid #dddddd15;;
    border-radius: 0 !important;
}
/* 选中文字背景 */
::selection {
    background-color: #FF69B475;
    color: #fff;
}

/* 右侧边栏宽度 */
.page-catalog-container {
    width: auto !important;
    min-width: 14em !important;
    max-width: 22em !important;
}
```
{% endnote %} 

## 推送到 Github 并设置 Pages

- `main` : 用于备份源文件
- `gh-pages` : 用于生成 Pages 的分支，由 Github Actions 自动构建

### 创建仓库
首先在 Github 上添加好公钥，然后建立一个空白的仓库。

{% note info %}
如果想使用 Github Pages，且以 `https://<username>.github.com/` 直接访问，而不要在链接中带有 repo 名称  
Repo 的名称应该设置为 `<username>.github.io` 的形式

否则链接为 `https://<username>.github.com/<repo>/`
{% endnote %}

### 设置 `.gitignore` 
创建 `.gitignore` 文件后，加入下面的内容 

```
node_modules
dist
.temp
.cache
```
### 初始化本地仓库

```sh
git init

git add --all
git commit -m "Initial Commit" --all
git branch -M main

git remote add origin https://github.com/<username>/<repo>.git
```

### 推送

```sh
git push -u origin main
```

## 设置 Github Actions

运行 `yarn docs:build` 会在 `.vuepress/dist` 下生成静态网页文件。

部署就是把 `dist` 文件夹的内容推送到 `gh-pages` 分支

使用 Github Actions 则不需要我们自己构建，可以实现当我们把源文件推送到 main 分支后自动构建并推送到 `gh-pages` 分支

### 设置 `workflows` 权限

- 参考： [VuePress - 部署 - Github Pages](https://v2.vuepress.vuejs.org/zh/guide/deployment.html#github-pages) 

首先，需要修改 `Github Actions` 的权限，不然最后 PUSH 时会报错，提示权限问题。

```
fatal: unable to access 'https://github.com/<username>/<repo>.git/': The requested URL returned error: 403
```
前往 Repo 的 `Settings` > `Action` > `General` > `Workflow permissions`，设置为 `Read and write permissions` 

![Action Permission](https://m.nep.me/blog/post/vuepress-github-actions-permissions.png)

### 创建 `workflows` 文件

需要创建 `.github/workflows/build-pages.yml` 文件

填入下面的代码

```yml
name: build-pages

on:
  # 每当 push 到 main 分支时触发部署
  push:
    branches: [main]
  # 手动触发部署
  workflow_dispatch:

jobs:
  docs:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3
        with:
          # “最近更新时间” 等 git 日志相关信息，需要拉取全部提交记录
          fetch-depth: 0

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          # 选择要使用的 node 版本
          node-version: 18
          
      - name: Run install
        uses: borales/actions-yarn@v4
        with:
          cmd: install # will run `yarn install` command

      # 运行构建脚本
      - name: Build VuePress site
        run: yarn docs:build

      # 查看 workflow 的文档来获取更多信息
      # @see https://github.com/crazy-max/ghaction-github-pages
      - name: Deploy to GitHub Pages
        uses: crazy-max/ghaction-github-pages@v3
        with:
          # 部署到 gh-pages 分支
          target_branch: gh-pages
          # 部署目录为 VuePress 的默认输出目录
          build_dir: docs/.vuepress/dist
        env:
          # @see https://docs.github.com/cn/actions/reference/authentication-in-a-workflow#about-the-github_token-secret
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

然后使用 git 追踪该文件

```
git add .github/workflows/build-pages.yml

git commit -m "create build-pages.yml"
```

在推送(`git push`)后可以访问 Repo 顶部的 Actions 查看运行状态

运行成功后我们可以查看 Repo 多了一个 `gh-pages` 分支

### 设置 gh-pages 作为 Source

前往 Repo 的 Settings，点击 Pages， 将 Source 选择为 `Deploy from a branch`
在 Brach 中选择 `gh-pages` ，点击 `save`

![Action Permission](https://m.nep.me/blog/post/vuepress-github-pages-source.png)

在 `Actions` 可以查看 `pages build and deployment` 的运行状态，完成后即可访问 

`https://username.github.io` 

查看


## 添加自定义域名

### 设置域名解析

为域名添加一条下面的解析

- 类型为 `CNAME` 
- 指向 `<username>.github.io`

如果使用的不是子域名， `www` 和 `@` 要一起解析

### Repo 添加文件
然后向 Github 的 Repo 内添加文件名为 `CNAME` 的文件，保证是大写

文件内容为所使用的域名
```
example.com
```

{% note info %}
CNAME 文件应该存在于在 `gh-pages` 分支，所以要前往 `.vuepress/public` 目录下创建 CNAME 文件
否则每次自动执行 `Github Actions` 之后，设置中的自定义域名设置都会重置
{% endnote %}

使用 CNAME 则不需要手动前往 `Settings` 设置自定义域名，等待 `Github Actions` 执行完成之后即可访问域名查看

### 添加自定义域名

前往 Repo 的 `Settings`，在 `Pages` 的 `Custom domain` 中填写自定义域名。 

点击 save 后，等待检查 DNS 生效就会保存成功。

保存成功后访问对应域名即可。

{% note info %}
域名是放在 Cloudflare 解析的话，如果 Github 提示解析出错，可以先暂停 Cloudflare 的 CDN
{% endnote %}