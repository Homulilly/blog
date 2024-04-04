---
title: OnePlus 12 - OxygenOS
date: 2024-04-02 15:45:14
tags:
 - Oneplus
categories:
 - Just-buy
---

最近突然又升起了换手机的欲望，之前一加 11 用了半年不是很满意就二手出掉了，所以原本是没有考虑造型基本一样的一加 12，不过这几天去商场转了一下，把国产手机摸了一圈。  
觉得一加 12 还不错，实机看起来并不丑。  
主要是这一波国产摄像模组的长焦更新体验确实非常好。   

记录一下摸过的国产手机们  
<!--more-->

- vivo x100 & x100 Pro  
由于准备入手 vivo Pad3 Pro，所以优先考虑了 vivo 的生态， x100 系列拍照非常的不错，不过实机体验边框宽了一些，前置摄像头的挖孔感觉也比其他家的大一圈。  
如果选的话，我会选 x100 ，Pro 的后置摄像头突出太高了，x100 就很舒服。  
网上看了看评价说是扬声器属于比较差的，所以减分 

- iQoo 12 & 12 Pro 
要换自然要选 8gen 3 或是 9300 的，第一次在一家店看 iQoo 12 Pro ，两侧曲边的绿边很明显，所以直接 Pass 不过可能是个例，第二家也见到过，没有那么明显。  
标准版是直屏，看起来还是可以，不过 iQoo 12 卖的并不便宜，比 x100 系列还要贵。  

- vivo X Fold 3 Pro
折叠机太贵了，而且外屏观感不好，个人感觉不如 OPPO 家的 FIND N3 的外屏，不过展开内屏看着还是很带感的。  
感觉没有必要。  

- OPPO Finx X7 & X7 Ultra  
不得不说，实机很是好看，后背的材质也是很舒服，看着非常有感觉，尤其是黑色中间还有织线，可惜 Ultra 太贵了，而且后置和 x100 Pro 一样隔手，标准版 X7 比较舒服一些。  

- OnePlus 12 
我是不太喜欢一加这个后置马桶盖的设计的，图片看起来就很不好看，手机壳也买不到好看的，所以之前用 11 的时候就感觉不爽，而且刚发售时还有大缝隙的质量问题，一开始根本没考虑 12。  
不过看 Find X7 时，顺带看了一眼，不得不说，白色实机看着还行，挺有感觉的。  
黑色的版本裸机手感好一些，但是摄像模组看起来丑。  
打开京东看了一眼，正好还在百亿补贴，不过白色的只补 16 + 1T 的版本，于是多掏了 300 块，反正我也是希望买大容量的。  
决定下单还一个比较重要的原因是，搜了一下发现一加 12 可以直接刷 OxygenOS 了，刷完后可以回锁 Bootloader 不会影响使用。  

稍微用了一下：
- 摄像：比 X7 和 x100 还是要差一些的，但够我使用的了。   
- 100w 快充是 OPPO 家私有协议，PD 只支持 18w，以及融合快充 33w。  
- 屏幕方面嘛，感觉是不如手上的 7T PRO 的，不过显示模式切换到生动后倒是和 7T Pro 的自然差不多了，所以感觉还可以，至少省去了适应的过程。  
- 重量：带壳正好半斤（ 重了一点  
 
## 刷 OxygenOS 14 

### 参考
 - [Oneplus 12 - How To switch from Color OS to Oxygen OS](https://tradingshenzhen.com/en/content/oneplus-12-how-to-switch-from-color-os-to-oxygen-os)
    - 我是看着这个教程刷的
    - 第四步需要按照教程提示在设备管理器中手动选择驱动
 - [https://0x20h.com/p/aa3f.html](https://0x20h.com/p/aa3f.html)
 - [XDA](https://xdaforums.com/t/how-to-convert-from-coloros-to-global-us-india-on-chinese-oneplus-12.4653255/)
 - [一加12 刷機 OxygenOS 14.0](https://www.mobile01.com/topicdetail.php?f=586&t=6920114)

### 刷机注意事项    
- 关机后，`电源` + `音量-` 进入 fastboot
- **刷完 OxygenOS 确保开机可以进入系统之后，再回锁 Bootloader**
- OxygenOS 的短信 APP 就是 Google 原生的了，不像国产的会区分通知短信与普通号码的短信。  

BL 重新上锁后可以正常使用指纹、面部解锁。

### BUG
**一加 12 刷 OxygenOS 后 Geekbench6 跑分过低**  
我兴冲冲的打开 Geekbench 6 结果 CPU 只跑了单核 900+ 多核 4000+，GPU 跑分倒是正常 14000+   
我一度怀疑我的 CPU 寄了，掏出我 7T Pro 的 855+ 都跑了 1000 分，搜了一下发现别人也有这种情况，还挺多。  
 - [FB](https://www.facebook.com/groups/459902981411036/permalink/1601667763901213/?_rdr) 
 - [Reddit](https://www.reddit.com/r/oneplus/comments/1akolqc/low_geekbench_score_fix_for_the_one_plus_12/)

Reddit 的帖子说是扩展内存默认设置的 4G 的问题，设置可以选 4/8/12 ，我关闭以及几个选项都尝试了，都是 900 来分。  
有评论说只是最新的系统(604)跑分 BUG -> [Reddit](https://www.reddit.com/r/oneplus/comments/1aihyjr/oneplus_12_whats_with_the_low_singlecore_scores/)。  

扩展内存开到 8G，再打开始终开启高性能模式，终于跑到了 2176 + 6612 ，还没到 2200 标准分（

本来用的嘎嘎流畅，跑分徒增痛苦。 

![Geekbenck6](https://m.nep.me/blog/post/oneplus12-gb6.jpg)