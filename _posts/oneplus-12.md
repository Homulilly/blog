---
title: OnePlus 12 与 OxygenOS 14 体验
date: 2024-04-02 15:45:14
tags:
 - Oneplus
categories:
 - Just-buy
---

最近突然又升起了换手机的欲望，之前一加 11 用了半年不是很满意就二手出掉了，所以原本是没有考虑造型基本一样的一加 12，不过这几天去商场转了一下，把国产手机摸了一圈。  
觉得一加 12 还不错，实机看起来并不丑。  
主要是这一波国产摄像模组的长焦更新体验确实非常好。   

记录一下在实体店摸过的国产手机们  
<!--more-->

## 体验过的机器  
### vivo x100 & x100 Pro  
由于准备入手 vivo Pad3 Pro，所以优先考虑了 vivo 的生态， x100 系列拍照非常的不错，不过实机体验边框宽了一些，前置摄像头的挖孔感觉也比其他家的大一圈。  
如果选的话，我会选 x100 ，Pro 的后置摄像头突出太高了，x100 就很舒服。  
网上看了看评价说是扬声器属于比较差的，所以减分 

### iQoo 12 & 12 Pro 
要换自然要选 8 gen3 或是 9300 的，第一次在一家店看 iQoo 12 Pro ，两侧曲边的绿边很明显，所以直接 Pass ,不过可能是个例，第二家也看了，没有那么明显。  
标准版是直屏，摸起来手感不错，不过 iQoo 12 卖的并不便宜，比 x100 系列还要贵。  
还有 iQoo Neo9 Pro 也是直屏的，但是摄像阉割太多。  

### vivo X Fold 3 Pro
折叠机太贵了，而且外屏观感不好，个人感觉不如 OPPO 家的 FIND N3 的外屏，不过展开内屏看着还是很带感的。  
总之感觉没有必要。  

### OPPO Finx X7 & X7 Ultra  
不得不说，实机很是好看，后背的材质也是很舒服，看着非常有感觉，尤其是黑色中间还有织线，可惜 Ultra 太贵了，而且后置和 x100 Pro 一样突出太高，隔手。  
标准版 X7 更加舒服一些。  

### OnePlus 12
我是不太喜欢一加这个马桶盖一样的后置摄像模组的设计，图片看起来就很不好看，手机壳也买不到好看的，之前用 11 的时候就感觉不爽，刚发售时还有大缝隙的质量问题，所以原本是没打算没考虑 一加 12 的。  
不过我在实体店看 Find X7 时，顺带看了一眼旁边的 OnePlus 12 ，不得不说，白色版本实机看着还行，非常的有感觉。    
虽然手感上摸起来不如黑色的舒服，但是现在手机都带壳使用了，好看才是第一位的，这里不得不说黑色的摸起来是真滴舒服。    
当场打开京东看一眼，居然正好在百亿补贴，不过白色的只补 16 + 1T 的版本，于是多掏了 300 块，反正我也是更偏向于买大容量的版本。  
我向直营店的销售人员出示了京东的价格，表示匹配不了，只能在线下单了。  
当然决定购买还一个比较重要的原因是，搜了一下发现 OnePlus 12 可以很轻松的刷 OxygenOS 了，刷完后可以重新锁上 Bootloader ，不会影响使用。  
到手后机器是 2023 年 12 月生产的，有些久了，估计是退换机，但是外观上并没有什么问题。后置摄像模组的缝隙已经处理好了，也没有胶水的样子。  
屏幕也不是阴阳屏。  
完美下车。  

用了几天的使用体验：
- 摄像方面听说是比不过 Find X7 和 x100 的，但足够我使用了，对焦很快，长焦也确实很牛。   
- 100w 快充是 OPPO 家私有协议，PD 只支持 18w，以及融合快充 33w。  
- 屏幕方面嘛，一开始感觉是不如手上的 7T PRO ，不过显示模式切换到生动后倒是和 7T Pro 的自然差不多了，没有色差，白色背景互相对比也不会显得不会偏绿，完美。  
- 重量：带壳正好半斤（ 是重了一点  
- OxygenOS：丝滑、流畅，非常的舒服。没有收不到通知的问题，完美。  
- 面容识别解锁是真滴又快又方便，2024 年了，我终于跟上时代了。
- OxygenOS 的短信 APP 就是 Google 原生的了，不像国产的会区分通知短信与普通号码的短信。但是现在基本都是通知短信了。也不是大问题。   
 
## 刷 OxygenOS 14 

### 参考
 - [Oneplus 12 - How To switch from Color OS to Oxygen OS](https://tradingshenzhen.com/en/content/oneplus-12-how-to-switch-from-color-os-to-oxygen-os)
    - 我是看着这个教程刷的
    - 第四步需要按照教程提示在设备管理器中手动选择驱动
 - [https://0x20h.com/p/aa3f.html](https://0x20h.com/p/aa3f.html)
 - [XDA](https://xdaforums.com/t/how-to-convert-from-coloros-to-global-us-india-on-chinese-oneplus-12.4653255/)
 - [一加12 刷機 OxygenOS 14.0](https://www.mobile01.com/topicdetail.php?f=586&t=6920114)

<img src="https://m.nep.me/blog/post/oneplus-12-about.jpg" alt="OnePlus 12" style="display: block; margin: auto; width: 50%;">

### 刷机注意事项    
- 关机后，长按 `电源` + `音量-` 进入 fastboot
- **刷完 OxygenOS 确保开机可以进入系统之后，再回锁 Bootloader**
- 目前看起来更新最快的是印度版的系统 
- BL 重新上锁后支付宝可以正常使用指纹与面部进行解锁、支付。

### 账号登陆问题
设置最顶部的账号登陆不了，一点击就会自动跳转到大陆的一加网站，然后 404 ，看着非常的不爽。  
可以访问网页版的 [Red Cabel Club - UK](https://www.oneplus.com/uk/redcableclub) 注册。  
昵称和头像也要在网页改，手机上 `设置` > `账号` > `一加会员` 修改会提示不能跨境使用。  
第一次登陆时挂个梯子吧。   

### 跑分 BUG
**一加 12 刷 OxygenOS 后 Geekbench6 跑分过低**  
vivo Pad3 Pro 跑出了 2258 + 7223 的分数后，我兴冲冲的打开手机 Geekbench 6 结果 CPU 只跑了单核 900+ 多核 4000+，这是 8 Gen3 吗？  
GPU 跑分倒是正常 14000+     
我一度怀疑我的 CPU 寄了，掏出我 7T Pro 的 855+ 单核都跑了 1000 分，搜了一下发现别人也有这种情况，还挺多。  
 - [FB](https://www.facebook.com/groups/459902981411036/permalink/1601667763901213/?_rdr) 
 - [Reddit](https://www.reddit.com/r/oneplus/comments/1akolqc/low_geekbench_score_fix_for_the_one_plus_12/)
 - [Your OnePlus Phone Can Be So Much Faster: Here's How](https://www.slashgear.com/1549879/one-plus-can-be-faster-heres-how/)
   看起来只有 OnePlus 12 有这种情况。  

Reddit 的帖子说是扩展内存默认设置的 4G 的问题，设置可以选 4/8/12 ，我关闭以及几个选项都尝试了，都是 900 来分。  
有评论说只是最新的系统(604)跑分 BUG -> [Reddit](https://www.reddit.com/r/oneplus/comments/1aihyjr/oneplus_12_whats_with_the_low_singlecore_scores/)，但我看有的评论说游戏也会卡。  

扩展内存开到 8G，再打开始终开启高性能模式，终于跑到了 2176 + 6612 ，还没到 2200 标准分（

本来用的嘎嘎流畅，只能说跑分徒增痛苦。 

<img src="https://m.nep.me/blog/post/oneplus-12-gb6.jpg" alt="OnePlus 12 Geekbenck6" style="display: block; margin: auto; width: 50%;">

## 问题

### 搜不到 5GHz 的 WIFI 

我有一个有些古老的 MikroTik RouterOS 路由器，一加 12 搜不到它的 5G 频段 WIFI ，我也是第一次碰到这个问题，其他设备都可以正常搜到这个 5G 频段的 WIFI。

但是一加 12 只能搜到 2.4G 频段。  

其他路由器的 WIFI6 目前都可以正常使用，WIFI7 我还没有见过。   

**解决方案**: 查看了一下 WIFI 的频道，默认是在 149(5745)，我调整为 36(5180) 后就可以搜到并加入了。 

大概与下面的有关，毕竟我的 OxygenOS 刷的是欧版的 ROM 
<img src="https://m.nep.me/blog/post/oneplus-12-wlan-channel.png" alt="WLAN Channel" style="display: block; margin: auto; width: 50%;">