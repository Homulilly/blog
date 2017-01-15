---
title: Ubuntu下数据恢复
tags:
  - Ubuntu
categories:
  - Linux
date: 2016-04-01 22:32:00
---

一次手滑引起的灾难。

装了个软件，想把生成的 .desktop 文件拷出来 于是  `cp /[path] ./~`  去 自己的 "~" 一看，没有，哦多加了个点。回去，重来，强迫症表示要删掉这个文件夹， `rm -r ~` 回答 y 灾难就是这么发生的。 （明明是一个文件，sad  

不过还好自己在感觉时间有点长的时候按下了 Ctrl+C ，大体浏览一下，/home/$user 下的文件夹都在，挂载的 /home/$user/downloads 目录已空。

其余的地方，我暂时也想不出来还有些什么文件。
<!--more-->
整体剩余的文件空间现在是80.5 GB剩余， 我也不记得原本有多少空间，大概是在10-20GB之间，还好主要都是一些动漫。

![Preview](/images/post/p02-preview.png)

于是，只能准备数据恢复了，在网上搜索了一下，有safebox， `sudo apt-get install safebox` ，呃，看了一下但是并不会用。

选择了另一个 extundelete ， 链接： [http://extundelete.sourceforge.net/](http://extundelete.sourceforge.net/)

<s>下载源码，`./configure --prefix=/opt/extundelete` 提示没有找到 ext2fs library , 搜了一下，需要安装 e2fslibs-dev ，apt-get 安装即可。 然后 `make && sudo make install` 。</s>

发现直接 `apt-get install extundelete` 即可

看了下网页上的提示 如果删除了文件，首先：

> $  `lsof|grep "/path/to/file"`
>
> progname 5559 user 22r REG 8,5 1282410 1294349 /path/to/file
> Notice the number in the second column is 5559 and the number in the fourth column is 22. The command to restore that file is:
>
> $ `cp /proc/5559/fd/22 restored.file`

然而，我这种删除了一大堆的，只能尽快将磁盘挂为已读或者卸载掉。
```bash
# If lsof doesn't find your file, then immediately remount the partition read-only:
$ mount -o remount,ro /dev/partition
# or unmount the partition:
$ umount /dev/partition

# or
$ dd bs=4M if=/dev/partition of=partition.backup

# 卸载磁盘后，

sudo /opt/extundelete/bin/extundelete /dev/sdb1 --restore-all -o /[path to put file]
```

但是提示  `extundelete: No such file or directory while creating directory` ，悲剧了，难道恢复不了？ 再找G娘，果然有和我相同遭遇的人，不过他说不用 `-o` 就好了，然后我就比较纳闷那恢复的文件会放在哪里？主目录还是当前目录？还是说直接在原磁盘恢复？不过最后一个想想也不可能。估计当前目录比较靠谱。

手动 cd 到移动硬盘，

> `sudo /opt/extundelete/bin/extundelete /dev/sdb1 --restore-all`  
> NOTICE: Extended attributes are not restored.  
> Loading filesystem metadata ... 739 groups loaded.  
> Loading journal descriptors ... 29260 descriptors loaded.  
> Searching for recoverable inodes in directory / ...  
> 4569 recoverable inodes found.  
> Looking through the directory structure for deleted files ...  
> Block 15040512 is allocated.  
> 200 recoverable inodes still lost.

终于开始恢复了，等了时间比较久，看最后一句话似乎并没有全部的恢复出来，打开恢复的文件夹一看，总共32.1GB，大概只有一半左右，也有部分文件是残缺的，没有完全恢复。恢复之前一不小心用了Chrome下载了一个文件，虽然不大，估计对数据恢复的影响应该不小。文件恢复的效果不是很好，有的图片虽然恢复出来的了，但是文件夹串了。所以恢复出来的数据，我也就取了里面的一个很完整的音乐文件夹，其余的意义不是很大了。

![2016-04-01 20-27-47.png](/images/post/p02-result.png)

还觉得比较怨念的是，这几天收的图都没有了，不过自己去翻Chrome下载列表的时候发现居然已经好几天没有清除下载列表了，于是手动了一大批的中键，差不多不用重复劳动了。

随后打算换一款软件再试试，看到了这篇文章 《[使用 Linux 文件恢复工具](https://www.ibm.com/developerworks/cn/linux/1312_caoyq_linuxrestore/)》。 试了一下Photorec，并不能以目录的形式恢复数据，同样参考数据的意义也不是很大。

总结，就是数据恢复的结果并不理想，可能是因为自己并没有立即的将磁盘设置为只读的原因。不过好在是没有什么重要的数据，动漫没了再下就是了，就是收图实在是麻烦一些。以后执行命令的时候只能更加的小心一些了。以不算太大的代价强化一下人生的经验，似乎也不算亏太多。
