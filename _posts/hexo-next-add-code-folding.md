---
title: Hexo NexT 主题添加代码折叠功能
date: 2023-04-04 16:17:37
tags: 
 - Hexo
categories:
 - Note
---

[Hexo Theme Icarus](https://github.com/ppoffice/hexo-theme-icarus) 的代码折叠功能很喜欢，奈何 NexT 主题不支持相应的样式，也[不打算支持](https://github.com/next-theme/hexo-theme-next/issues/178#issuecomment-759120606)，推荐的是使用 NOTE 块的实现方式，这很不优雅。 

今天我发现在 NexT 主题中使用下面的格式时，代码块也会生成一个标题栏。
````md file or summary 
```js file or summary
/* some code */
```
````

那实现起来就很简单了，只需要掏出 F12 ，在 `file or summary` 前面插入一个按钮，然后为按钮添加功能。  
<!--more-->
不会写 JS 没有关系，只需要咨询一下 ChatGPT 。

{% note info %}
Hexo v5.4.2
NexT v8.15.0
{% endnote %}

## 配置主题加载自定义内容
在主题配置文件 `_config.yaml` 中，找到 `custom_file_path`，取消对应的注释，允许加载自定义内容
```yaml themes/next/_config.yaml
custom_file_path:
  style: source/_data/styles.styl
  bodyEnd: source/_data/body-end.njk
```

然后在 Hexo 主目录下，创建对文件文件。

## 添加代码

在 `body-end.njk` 文件中，添加 JS 代码
```js source/_data/body-end.njk
<script>
    document.addEventListener("DOMContentLoaded", function() {
      // 查找所有匹配的 span 标签
      const spans = document.querySelectorAll("figcaption > span");
  
      // 遍历所有匹配的 span 标签
      spans.forEach(function(span) {
        // 在 span 标签前插入 <i> 标签
        const iElement = document.createElement("i");
        iElement.className = "fas fa-angle-down";
        // 插入一点空格
        iElement.innerHTML = "&nbsp;&nbsp;";
        span.parentNode.insertBefore(iElement, span);
  
        // 查找相邻的 <div class="table-container">
        const tableContainer = span.parentElement.nextElementSibling;
  
        // 为 <i> 标签添加点击事件
        iElement.addEventListener("click", function() {
          // 切换 tableContainer 的 "hidden" 类
          tableContainer.classList.toggle("code-hidden");
  
          // 切换 <i> 标签的类名
          if (iElement.classList.contains("fa-angle-down")) {
            iElement.classList.remove("fa-angle-down");
            iElement.classList.add("fa-angle-right");
          } else {
            iElement.classList.remove("fa-angle-right");
            iElement.classList.add("fa-angle-down");
          }
        });
      });
    });
  </script>
```

在 `styles.styl` 填写对应的样式代码
```css source/_data/styles.styl
/* 代码块隐藏 */
.code-hidden {
    display: none;
  }
```

加入后，运行本地预览 `hexo g && hexo s`，效果非常完美。

现在的问题是，如果代码块不写 `file or summary ` 内容的话，是没有这个 `<figcaption>` 标题栏的。 

## 为大于三行的代码块添加标题栏

一两行的代码也没有必要进行折叠，所以添加一个检测，为大于三行且没有标题栏的代码块手动添加一个，最终代码如下

```js source/_data/body-end.njk
<script>
  document.addEventListener("DOMContentLoaded", function() {
    // 查找所有 div.table-container 元素
    const tableContainers = document.querySelectorAll(".table-container");

    // 遍历所有 div.table-container 元素
    tableContainers.forEach(function(tableContainer) {
      // 获取 div.table-container 内的 span 元素数量
      const spanCount = tableContainer.querySelectorAll("tbody > tr > td.code > pre > span").length;

      // 检查 span 元素数量是否大于等于 3
      if (spanCount >= 3) {
        // 检查 div.table-container 前面是否有 figcaption 元素，如果没有则添加一个
        const prevElement = tableContainer.previousElementSibling;
        let figcaption;
        let iElement;
        if (!prevElement || prevElement.tagName.toLowerCase() !== "figcaption") {
          // 在 div.table-container 前插入一个 figcaption 元素
          figcaption = document.createElement("figcaption");

          // 将 figcaption 插入到 DOM 中
          tableContainer.parentNode.insertBefore(figcaption, tableContainer);
        } else {
          figcaption = prevElement;
        }

        // 创建一个 <i> 标签并添加功能
        iElement = document.createElement("i");
        iElement.className = "fas fa-angle-down";
        // 插入一点空格
        iElement.innerHTML = "&nbsp;&nbsp;&nbsp;";
        figcaption.insertBefore(iElement, figcaption.firstChild);

        // 为 <i> 标签添加点击事件
        iElement.addEventListener("click", function() {
          // 切换 tableContainer 的 "code-hidden" 类
          tableContainer.classList.toggle("code-hidden");

          // 切换 <i> 标签的类名
          if (iElement.classList.contains("fa-angle-down")) {
            iElement.classList.remove("fa-angle-down");
            iElement.classList.add("fa-angle-right");
          } else {
            iElement.classList.remove("fa-angle-right");
            iElement.classList.add("fa-angle-down");
          }
        });
      }
    });
  });
</script>
```

然后预览一下，看一下效果，没有问题的话就可以推送到服务器了。