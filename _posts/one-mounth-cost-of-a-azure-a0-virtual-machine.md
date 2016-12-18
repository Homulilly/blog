---
title: Azure A0 虚拟机月消费
date: 2016-03-25 18:08:00
tags:
---

话说都说软娘的Azure非常的不错。  
然而看了下价格，个人用的话真是略昂贵，而且似乎什么都是收费项目。  
微软提供了免费使用，给了不少的免费月消费额度，于是自己测试了一下最便宜的 A0 级别的虚拟机一个月的消费。   
因为是2月份，所以计费周期只有 29 天。  

A0 机器的配置：  
0.25颗核心 / 0.75GB RAM / 20GB磁盘  
CPU 为 Intel Xeon CPU E5-2673 v3 @ 2.394GHz  
RAM Ubuntu显示总计668MB  
位置为亚洲东部（HK）  
<!--more-->
下面是2月份29天账单周期的总消费及详细信息：  
总计 137.5 HKD   

机器就跑了当前的 Typecho， 使用的是 Apache2 + Mysql 。  

1. Networking - Data Transfer In (GB) - Zone 2 （亚太、日本、澳大利亚）  
流入流量：使用 0.261609 GB，费率 0，虽然有统计，但是并不计费。  
而且看来执行系统更新的流量也不会被统计。   

2. Networking - Data Transfer out (GB) - Zone 2  
流出流量：使用 0.371572 GB，费率约为 1.077 ， 开销 0.40 HKD  
于是，是一个毫无存在感的站点。  

3. Data Management - Storage Transactions (in 10,000s)  
计费单位 10,000s，使用71.5184，开销 0.20HKD  
看起来像是一个抢钱的项目，不过费率非常的低，所以影响也不大。  

4. Virtual Machines - A0 VM Compute Hours - AP East  
虚拟机：使用679.78小时，费率约0.1552，总开销 105.50 HKD  
大头项目。  

5. Networking - Public IP Addresses   
固定公网IP：使用682小时，费率约0.031，总开销 21.17 HKD  
固定IP的开销也不少，不过这里出现了BUG，我说先开的机器才知道默认分配的并不是固定IP，动态分配的IP倒是是免费的，然后才分配了这个固定IP，但是使用时间上反而是IP的使用时间比机器的时间要长。   

6. Storage - Standard IO - Table/ Queue (GB)   
计费单位GB使用约0.0029，费率0， 不计费。  

7. Storage - Standard IO - Page Blob/Disk (GB)   
计费单位GB，使用约26.38，费率约0.3878，总开销 10.23 HKD  
这感觉也是一个抢钱的项目，而且开销比重还不小。  

于是一个月在没有什么流量的情况下开销达到了 137.5 HKD  。  
还是性能最差的A0机器。  
A1机器的开销要大概是A0机器的三倍，具体可以通过软娘提供的定价计算器计算 <https://azure.microsoft.com/zh-cn/pricing/calculator/>  

总之，感觉就是害怕，关于访问稳定性方面，浙江电信这边的 ping 值在电信不抽风的时候在40ms左右，不过这是在地理位置来说蛮正常的，测试过别人的HK站点，也差不多就是这样，电信抽风的时候，当然谁也跑不了。  
