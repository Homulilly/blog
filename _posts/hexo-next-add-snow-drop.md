---
title: Hexo NEXT 主题添加雪花飘落的效果
date: 2023-03-24 22:39:11
tags:
 - Hexo
categories:
 - Note
---

主题的模板改成 NexT.Pisces 后，背景看着有些单调，就想加点特效。

搜索一番，发现现在 NexT 主题可以很方便的引入自定义内容了，不需要修改主题的源文件，方便多了。   
一通鼓捣，加上了雪花飘落的效果。  

下面是具体的添加方法。

<!--more-->

{% note info %}
Hexo v5.4.2
NexT v8.15.0
适用于深色主题
{% endnote %}

## 修改主题设置

使用了 Javascript 和 CSS 来实现效果

在 NexT 主题的 `_config.yaml` 中，找到 `custom_file_path` ，取消对应的注释
```yaml
custom_file_path:
  # 存放自定义 CSS 样式
  style: source/_data/styles.styl
  # 存放自定义的 Javascript 脚本
  bodyEnd: source/_data/body-end.njk
```

然后在 Hexo 目录下的 `source` 文件夹，创建 `_data` 文件夹
再创建 `styles.styl` 和 `body-end.njk` 文件

## 添加实现代码

### 添加 JS 代码

在 `body-end.njk` 中加入 JS 代码

- 雪花随机大小
- 有三种样式
- 落到底部会堆叠一段时间

```js
<script>
    let snowflakeInterval;

    function createSnowflake() {
        const snowflake = document.createElement("span");
        snowflake.classList.add("snowflake");

        // 生成不同的雪花
        const randomSnow = Math.random();
        if (randomSnow < 0.1) {
            snowflake.textContent = "⛄";
        } else if (randomSnow < 0.6){
            snowflake.textContent = "❄";
        } else if (randomSnow < 0.8){
            snowflake.textContent = "❅";
        } else if (randomSnow < 0.9){
            snowflake.textContent = "❉";
        } else {
            snowflake.textContent = "✥";
        }

        snowflake.style.left = `${Math.random() * 100}vw`;
        snowflake.style.top = '0px';

        // 设置雪花生成大小和透明度
        const size = Math.random() * 18 + 10;
        snowflake.style.fontSize = `${size}px`;

        const snowopcity = Math.random() * 0.6;

        if (size > 18){
            snowflake.style.opacity = snowopcity + 0.4;
        }else{
            snowflake.style.opacity = snowopcity;
        }
        

        // 设置下坠速度 越小越快
        const translateYDuration = Math.random() * 20 + 5;
        const translateXDuration = Math.random() * 5 + 2;
        // 设置旋转速度
        const rotationDuration = Math.random() * 3 + 1;

        let startTime = null;

        // 增加堆积时间
        const accumulateDuration = 10;

        function update(timestamp) {
            if (!startTime) startTime = timestamp;

            const progress = (timestamp - startTime) / 1000;
            let translateY = (progress / translateYDuration) * 2000;
            const translateX = Math.sin((progress / translateXDuration) * Math.PI) * 100;
            const rotation = (progress / rotationDuration) * 360;

                // 当雪花到达窗口底部时，停止下落并堆积
                if (translateY + snowflake.offsetHeight > window.innerHeight) {
                    translateY = window.innerHeight - snowflake.offsetHeight;

                // 如果堆积时间已经过去，则删除雪花
                    if (progress > translateYDuration + accumulateDuration) {
                        snowflake.remove();
                        return;
                    }
                }

            snowflake.style.transform = `translateY(${translateY}px) translateX(${translateX}px) rotate(${rotation}deg)`;

            if (progress < translateYDuration + accumulateDuration) {
                requestAnimationFrame(update);
            } else {
                snowflake.remove();
            }
        }

        requestAnimationFrame(update);
        document.body.appendChild(snowflake);
    }

    
    function startSnowfall() {
        // 载入时若边栏是隐藏状态则不加载雪花
        const sidebarnav = document.querySelector('.sidebar');
        const sidebarnavdisplay = window.getComputedStyle(sidebarnav).getPropertyValue('display'); 
        if (sidebarnavdisplay !== 'none') {
            // 设置雪花生成速度 数字越小越生成越快
            snowflakeInterval = setInterval(createSnowflake, 350);
        }
    }

    function stopSnowfall() {
        clearInterval(snowflakeInterval);
    }

    function handleVisibilityChange() {
        if (document.visibilityState === "visible") {
            startSnowfall();
        } else {
            stopSnowfall();
        }
    }

    document.addEventListener("visibilitychange", handleVisibilityChange);

    // 初始化时，如果页面可见，则开始生成雪花
    if (document.visibilityState === "visible") {
        startSnowfall();
    }
</script>
```

{% note info 不带底部堆叠的代码 %}
```js
<script>
    let snowflakeInterval;

    function createSnowflake() {
        const snowflake = document.createElement("span");
        snowflake.classList.add("snowflake");

        // 10% 概率生成雪人（⛄），90% 概率生成雪花（❄）
        snowflake.textContent = Math.random() < 0.1 ? "⛄" : "❄";

        snowflake.style.left = `${Math.random() * 100}vw`;
        snowflake.style.top = '0px';

        // 设置雪花生成大小和透明度
        const size = Math.random() * 18 + 12;
        snowflake.style.fontSize = `${size}px`;
        snowflake.style.opacity = Math.random() * 0.6 + 0.3;

        // 设置下坠速度 越小越快
        const translateYDuration = Math.random() * 20 + 5;
        const translateXDuration = Math.random() * 5 + 2;
        // 设置旋转速度
        const rotationDuration = Math.random() * 3 + 1;

        let startTime = null;

        function update(timestamp) {
            if (!startTime) startTime = timestamp;

            const progress = (timestamp - startTime) / 1000;
            const translateY = (progress / translateYDuration) * 2000;
            const translateX = Math.sin((progress / translateXDuration) * Math.PI) * 100;
            const rotation = (progress / rotationDuration) * 360;

            snowflake.style.transform = `translateY(${translateY}px) translateX(${translateX}px) rotate(${rotation}deg)`;

            if (progress < translateYDuration) {
                requestAnimationFrame(update);
            } else {
                snowflake.remove();
            }
        }

        requestAnimationFrame(update);
        document.body.appendChild(snowflake);
    }

    
    function startSnowfall() {
        // 载入时若边栏是隐藏状态则不加载雪花
        const sidebarnav = document.querySelector('.sidebar');
        const sidebarnavdisplay = window.getComputedStyle(sidebarnav).getPropertyValue('display'); 
        if (sidebarnavdisplay !== 'none') {
            // 设置雪花生成速度 数字越小越生成越快
            snowflakeInterval = setInterval(createSnowflake, 350);
        }
    }

    function stopSnowfall() {
        clearInterval(snowflakeInterval);
    }

    function handleVisibilityChange() {
        if (document.visibilityState === "visible") {
            startSnowfall();
        } else {
            stopSnowfall();
        }
    }

    document.addEventListener("visibilitychange", handleVisibilityChange);

    // 初始化时，如果页面可见，则开始生成雪花
    if (document.visibilityState === "visible") {
        startSnowfall();
    }
</script>
```

```js
snowflake.textContent = Math.random() < 0.1 ? "⛄" : "❄";
```
这里设置了 10% 会生成一个 ⛄ ，如果只想生成一种可以使用下面的代码

```js
// 设置成其他字符也是可以的
snowflake.textContent = "❄"
```
{% endnote %}

### 添加样式

在 `styles.styl` 文件中加入雪花样式代码，
- 设置颜色
- 置于网页底部层级
- 鼠标无法选择
- 加入一点荧光

```css
/* 雪花 */
.snowflake {
    position: fixed;
    color: white;
    display: block;
    position: fixed;
    pointer-events: none;
    z-index: -1;
    text-shadow: 0 0 5px rgba(255, 255, 255, 0.8), 0 0 10px rgba(255, 255, 255, 0.8);
}
```

此时可以 `hexo g && hexo s` 预览了。

不过这时可以看到由于 NexT.Pisces 边栏和文章主体部分没有透明效果，效果并不好。

我们可以在 `styles.styl` 中加入下面内容，加上一点透明效果。

```css
/* 设置左侧边栏透明 */
.sidebar, .header, {
    opacity: 0.85;
}

/* 设置文章部分透明 */
.main-inner {
    opacity: 0.95;
}
```

本地预览没有问题，既可以推送到服务端了。

![:2233_卖萌:](https://m.nep.me/emoji/2233/9fdb0139c2e2a925eba87cd135b49c17eb08ba30.png)