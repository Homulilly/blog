---
title: 入手了一个 MBP，但上了 SONY 的当
date: 2024-11-19 01:03:46
tags:
 - MacBook Pro
 - Apple
 - SONY
 - INZONE M10S
categories:
 - Just-buy
---

这苹果的 M4 看着是风生水起，搞的我有一点心动，正好手上的笔记本 ASUS Zenbook Duo 感觉越来越不行了，两块屏幕的色差感觉比刚买的时候强烈多了。  

挑选了一番，最后入手了 M4 Pro 的 MBP 。

至于关于索尼的故事，由于觉得笔记本屏幕效率太低，干活有点不得劲，打工人打算给自己升级一下工具。  

原本我是纠结买 27 寸 4K 还是 32 寸 4K，于是就去商场实际体验一下，故事就是从这里开始。   

<!-- more -->

逛商场的时候，我看到了大法新出的 INZONE 系列显示器，说实话之前 INZONE M9 打折时的想入手的来着，但是没货，没有买到，略有遗憾。   

然后就看到了主角，**INZONE M10S**，2K OLED 480Hz 专业 FPS 电竞显示器，小弟 **INZONE M9 II** 倒是使用的是 4K 分辨率，但是是 IPS 屏幕，看起来普普通通。  

说实话，商场里 M10S 放的展示视频，素质是真的好，非常的让我心动。回家后，思考一番，回去买了下来。  

人生苦短，及时享乐。  

然后惨遭暴击：

1. 这索尼，这么贵的显示器，居然只给一根 DVI ，不给 HDMI 的线，导致我开箱后一脸懵，毕竟我现在只有 HDMI 的输出设备，就只能看着它，什么也干不了，天色已晚，出门也没有买到，只能网上下单一根，等第二天送货。 
2. 然后又由于我 Mac 也是第一次用，不知道 macOS 对 2K 分辨率基本等于没有高清（HiDPI）支持，非常的难用，不过有第三方软件 Better Diskplay 可以稍稍拯救一下。
3. 这是电竞显示器，所以显然也没有 TYPE-C 输入，MBP 插着一根 HDMI 线在右边，很不优雅，不过这我倒是提前就接受了。  

只能说是又上了索尼的贼船，拿出来水水文章。

至于连接效果，通过 HDMI 连接，2K 480 Hz 是支持的，看着可以动态 24 - 480。

## MBP 

关于 MBP 的本体，这我倒是很满意，很重要的一点就是屏幕素质确实一流，看起来很舒服。   

不爽的就是内存和磁盘太过金贵，有点费米。  

从 Windows 转过来，上手还是不太习惯，记录一下问题与解决方案。  

### 屏幕分辨率

首先就是屏幕分辨率部分，2K 原生看起来确实很难看，完全没有优化，甚至还不如 Windows 显示的，苹果针对 2K 提供的 HiDPI 选项是 720P ，就是实际现实 720P 屏幕呈现的内容，太小了，更难用。  

不过可以安装第三方软件 [Better Display](https://github.com/waydabber/BetterDisplay) 强制开启其他档位的 HiDPI 选项，我开了 2048x1152 ，看着还可以接受。  

主要我没用过 4K 的版本，没有太大落差，毕竟 27 寸 2K PPI 就摆在那里，只有 108 ，27 寸 4K 则有 163，32 4K 是 138。 

不过 PPI 不够，可以距离来凑，我发现在大约 75cm 视距时效果很不错，还能保持良好坐姿。  

我最后还是换了个 32 寸 4K ，毕竟高刷对我来说 120Hz 就足够了 ， 毕竟 **大就是好** ，而且有系统原生支持的 4K 缩放优化显示效果就是好了不少。

关于刷新率，480 和 120 基本也就是能从晃动鼠标的轨迹看出来。

至于 M10S 吧，显示效果还是很好的，尤其是作为 OLED 侧面看显示效果完全没有衰减，非常惊艳（但是好像没有太大用处），如果是 4K 就好了。  
而且 OLED 不会漏光，显示的黑色时 IPS 没法比的。

### 鼠标滚轮

滚轮和 Windows 滚动方向是相反的，系统设置中可以找到鼠标选项，取消自然滚动，但是触摸板也会同步修改。  

有第三方软件 [Scroll Reverser](https://pilotmoon.com/scrollreverser/) 可以解决，将鼠标滚轮与触摸板的滚动方向分开设置。

或是使用 [MOS](https://mos.caldis.me/) ，这个浏览器鼠标滚动更顺滑一些。

### 无法修改头像

我给账户换个头像，一会就回给我换回棒球。  

但是我登录网页管理，看着头像又是新换的。  

可能是由于我是先创建了本地账户然后登陆 Apple ID 导致的，前往 **系统设置** > **用户与群组**， 修改本地账户的头像。

### 键位

Control、Option、Command 与 Ctrl、Alt、Windows 对应，但是位置不同。  

尤其复制粘贴是 Command + C/V ，对应的是 Windows 键。

还有切换输入法是 Caps Lock 键。

系统设置可以调整键位映射，让我先适应一下，看看可不可以。

不过关于顺序问题，可以前往 **系统设置** > **键盘** > **键盘快捷键** > 左侧再点 **修饰键** ，调整一下对应键盘设置，然后扣一下键帽，调整一下。

还想吐槽的是 FN 在最外侧的都不是好设计。  

### 待机后再点亮无法检测到外接屏幕

我发现如果待机久了，回来就会无法检测外接屏幕。

手动切换一下输入，或是在显示器中将自动切换输入改为固定的 HDMI 1。  

### INZOME M10S 自动降低亮度

用了第二天我发现，在 M10S 上白色背景的网页或是图片，如果全屏亮度就会降低。  

试了一下 Windows 上也会复现，一番搜索，原来是 OLED 的 ABL 功能。

>ABL：全称 Automatic Brightness Limiter，自动亮度限制。
>ASBL：全称 Automatic Static Brightness Limiter ，自动静态亮度限制。

看起来无法关闭，只能调低显示器亮度使用，或是使用深色模式。

不是很喜欢深色模式，发现 OSD 菜单中有亮度稳定的选项，开启后，限制了最高亮度，但是不会出现变化导致影响观感。  


### 安装 Stable Diffusion

#### 推荐 MochiDiffusion

[MochiDiffusion](https://github.com/MochiDiffusion/MochiDiffusion/releases)

在 Releases 中下载 dmg 直接安装即可。 

不过需要自行转换或下载 Core ML 模型，[下载](https://huggingface.co/coreml-community) 已转换的模型。

下载后，放入用户 Home 目录下 `MochiDiffusion/models` 文件夹中。··

#### 或是安装原版 Stable Diffusion WebUI
先安装 [Homebrew](https://brew.sh/)

安装必要依赖
```sh
brew install cmake protobuf rust python@3.10 git wget
```

Clone **Stable Diffusion WebUI**
```sh
cd ~
git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui
```

运行
```sh
cd stable-diffusion-webui

./webui.sh
```

Stable Diffusion WebUI 会自动使用 python3.10
---
