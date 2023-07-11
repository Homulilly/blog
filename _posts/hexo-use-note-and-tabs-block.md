---
title: 在 Hexo 中使用 NOTE 提示块 
date: 2023-03-22 17:00:16
tags:
 - Hexo
categories:
 - Note
---

在 Hexo 中可以插入一些很酷的提示块，效果如下：

## 1. 基本语法

```
{% note success %}
一个 Success 提示
{% endnote %}
```

{% note success %}
一个 Success 提示
{% endnote %}

<!--more-->

提示块可以实现折叠   
````
{% note success 一个折叠的提示 %}
一个折叠的 Success 提示
{% endnote %}
````
{% note success 一个折叠的提示 %}
一个折叠的 Success 提示
{% endnote %}

## 2. 更多风格

下面是全部支持的样式

{% note default %}
一个 Default 提示
{% endnote %}

{% note primary %}
一个 Primary 提示
{% endnote %}

{% note success %}
一个 Success 提示
{% endnote %}

{% note info %}
一个 Info 提示
{% endnote %}

{% note warning %}
一个 Warning 提示
{% endnote %}

{% note danger %}
一个 Danger 提示
{% endnote %}

对应的代码

```
{% note default %}
一个 default 提示
{% endnote %}

{% note primary %}
一个 primary 提示
{% endnote %}

{% note success %}
一个 success 提示
{% endnote %}

{% note info %}
一个 info 提示
{% endnote %}

{% note warning %}
一个 warning 提示
{% endnote %}

{% note danger %}
一个 danger 提示
{% endnote %}
```

## 3. Hexo NEXT 中设置样式

当前使用的 `flat` 的风格 
```yaml
# Note tag (bootstrap callout)
note:
  # Note tag style values:
  #  - simple    bootstrap callout old alert style. Default.
  #  - modern    bootstrap callout new (v2-v3) alert style.
  #  - flat      flat callout style with background, like on Mozilla or StackOverflow.
  #  - disabled  disable all CSS styles import of note tag.
  style: flat
  icons: false
  # Offset lighter of background in % for modern and flat styles (modern: -12 | 12; flat: -18 | 6).
  # Offset also applied to label tag variables. This option can work with disabled note tag.
  light_bg_offset: 0
```

## 4. 使用 Tabs 标签

还可以在文章内设置选项卡

{% tabs 标签, 1 %} 每个子标签显示为 `标签 + 数字序号` ，默认展开第 1 个选项卡，如果是 -1 则只显示标签，不显示内容，点击可以显示内容
<!-- tab -->
**选项卡 1** 
<!-- endtab -->
<!-- tab -->
**选项卡 2**
<!-- endtab -->
<!-- tab TAB 三 -->
**选项卡 3** 名字为 `TAB 三`
<!-- endtab -->
{% endtabs %}

下面是上面标签的代码

```
{% tabs 标签, 1 %} 
<!-- tab -->
**选项卡 1** 
<!-- endtab -->
<!-- tab -->
**选项卡 2**
<!-- endtab -->
<!-- tab 标签三 -->
**选项卡 3** , 名字为 `TAB三`
<!-- endtab -->
{% endtabs %}
```
第一行 `{% tabs 标签, 1 %}` 设置每个子标签在不指定标题时显示为 `标签 + 数字序号`
`1` 表示默认展开第 1 个选项卡，如果是 `-1` 则只显示标签标题，隐藏内容，点击可以显示内容

\ Awesome! /

**Ref:** [在hexo-NexT中插入note提示块](https://jinnsjj.github.io/uncategorized/hexo-next-note/)
