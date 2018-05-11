---
title: Hexo Next 主题点击加载 Disqus
date: 2016-10-25 18:20:32
tags:
 - Hexo
categories:
 - Note
---
感觉 Disqus 的评论载入后有点花 ，发现别人有的点击载入方式很不错，搜索了一下，只要简单的 Copy & Save 就能搞定，于是就 `ヾ(●゜▽゜●)` 。

需要修改的文件：
- `[HEXO_PATH]/themes/next/layout/_partials/comments.swig`
- `[HEXO_PATH]/themes/next/layout/_scripts/third-party/comments/disqus.swig`
- `[HEXO_PATH]/themes/next/source/css/_custom/custom.styl`

<!--more-->

### 1. 添加按钮
编辑第一个 `comments.swig` 文件，在文件内容 `<div id="disqus_thread">` 前面加入下面内容：
```html
<div style="text-align:center;">
  <button class="btn" id="load-disqus" onclick="disqus.load();">加载 Disqus 评论</button>
</div>
```

Update:
一开始没有发现 Next 自带了一个按钮的样式，没有加上 `class="btn"`，所以在第三步中自己写了一个简单的样式，不过自带的感觉更好一些，还有动画什么哒～

### 2. 替换 Disqus 载入的 js

然后编辑 `disqus.swig` 文件，需要替换原本的 Disqus 的加载的内容，如果希望显示评论数量，就保留 `run_disqus_script('count.js')` 这一行，这样页面载入时还会加载 disqus 的资源：
```js
run_disqus_script('count.js');
{% if page.comments %}
  run_disqus_script('embed.js');
{% endif %}
```

替换为下面的内容。
```js
var disqus = {
  load : function disqus(){
      if(typeof DISQUS !== 'object') {
        (function () {
        var s = document.createElement('script'); s.async = true;
        s.type = 'text/javascript';
        s.src = '//' + disqus_shortname + '.disqus.com/embed.js';
        (document.getElementsByTagName('HEAD')[0] || document.getElementsByTagName('BODY')[0]).appendChild(s);
        }());

        $('#load-disqus').remove(); ///加载后移除按钮
      }
  }
}
```

前面的 `function run_disqus_script(disqus_script){}` 这一段，不打算显示评论数量的话，可以一起删掉，不显示评论数量的话，那么点击加载按钮之前，网页是不会加载来自 Disqus 的资源的。

### 3. 为按钮添加样式
然后在 `custom.styl` 文件内给需要点击的按钮添加样式，第一步中加了 `class="btn"` 的话，就不用管这一段了 ：
```css
// for comment load button
#load-disqus {
    background-color:#FFFFFF;
    padding:3px;
    margin:0 auto;
    position:relative;
    font-size:14px;
    color:#555;
    text-decoration:none;
    border: solid 1px #2D2D2D;
}
```

### 4. Problem
1. 如果选择不显示评论数量的话，Next 主题所带的显示评论数目地方的地方都只显示一个竖杠就结束了，感觉不太好，在 `custom.styl` 加入 css 屏蔽掉好了:
  ```css
  .post-comments-count{
    display: none!important;
  }
  ```
2. 网站有新评论的话，Disqus 默认是没有通知的，找了一下也只有邮件的通知，打开就好了。
3. 感觉原本 Next 应该是优先载入评论的，文章都是页面刷出来后等一下才载入，于是那个点击加载的按钮在页面刷出来的时候就存在了，网页拉到底部刷新一下就能发现了，不过除非是太短的文章，也没有啥影响的样子。  

### 5. Update  
更新 Next 到最新的 v5.1.3 后，按钮的样式不太正常。看着是只有鼠标悬浮的样式，不想费劲找那里出了问题。简单粗暴的处理了一下。  


将第一步中的 `<button class="btn"` 修改为 `<button class="disbtn"`  
然后直接将对应的 css 样式粘帖到 `custom.styl` 中就可以了。  

```
.disbtn {
    display: inline-block;
    padding: 0 20px;
    font-size: 14px;
    color: #fff;
    background: #222;
    border: 2px solid #222;
    text-decoration: none;
    border-radius: 0;
    transition-property: background-color;
    transition-duration: 0.2s;
    transition-timing-function: ease-in-out;
    transition-delay: 0s;
    line-height: 2;
}

.disbtn:hover {
  border-color: #222;
  color: #222;
  background: #fff;
}
```

参考链接：
- [Disqus点击加载](https://gist.github.com/JimmehCai/84dead2d2919af05fede)
- [如何加载Disqus onclick事件点播](http://zh-cn.affdu.com/how-to-load-disqus-onclick-event.html)
