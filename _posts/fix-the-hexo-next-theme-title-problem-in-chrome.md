---
title: 修复 Hexo Next 主题在 Chrome 下标题换行的问题
date: 2016-10-23 22:59:40
tags:
 - Hexo
---
这 Next 主题用着不错，也基本没碰到啥问题，不过有时文章的标题在首页时会在最后一个字换行，强迫症表示忍了很久还是忍不住了。  

效果是下面这样的， <s>触发原因不明，看了下有三个出现这种情况的单词，都是英文 + 1个数字 + 中文，但是明明有比这三个长很多的都可以正常显示，所以也不确定。</s>  

![next_theme_title.png](/images/post/p05-next-theme-title.png)

试了一下，发现是因为字体的原因，我在主题配置文件中将全局字体 `Lato` 修改为了 `Ubuntu`，试了试，确实是这个原因。

Firefox 下看了没有这个问题，别的浏览器没有试。  
于是戳了戳 Chrome 的 F12，很快就找到影响这个的 css

```css
/*in main.css?v=5.0.1:1226*/
.posts-expand .post-title-link {
    display: inline-block;
    ...
}
```

编辑 `[Path of Hexo]/themes/next/source/css/_common/components/post/post-title.styl` 文件， 把 `display: inline-block;` 删掉或者是用 `/* */` 注释掉，然后就可以了。  

暂时，看了一圈，没看到啥页面出现 BUG 或是和以前不同的地方。

---
