---
title: Ubuntu本机迁移磁盘
tags:
  - Ubuntu
categories:
  - Linux
date: 2016-06-04 16:14:00
---

台式机上之前把Ubuntu装到了机械硬盘上，最近从旧本子上拆下了之前使用使用的SSD。

犹豫不想重装，打算直接克隆迁移一下，搜索一下，还是可行的。

准备：写一个 Ubuntu 的 LiveCD 就可以了。

参考文章 ： [将Arch Linux系统迁移到SSD](http://www.longxk.com/posts/2014/03/10/migrate-arch-install-to-ssd/)
<!--more-->
对SSD磁盘进行分区：
```bash
#检查分区情况  
sudo fdisk -l  
#分区
sudo fdisk /dev/sdc #要迁移到的 SSD 磁盘为 sdc
#如果需要 swap ，在 fdisk 中用“t”命令将新添的分区id改为82（Linux swap类型）
#格式化
sudo mkfs -t ext4 -c /dev/sdxx
sudo mkswap /dev/sdxx
```

迁移文件（这个和参考的文章完全一样就可以了）：

机械硬盘：

    sudo mount -o rw,relatime,data=ordered /dev/sdxx /mnt/oldroot

固态硬盘： （如果 SSD 支持 TRIM 指令，在挂载时加上一个 discard 选项可以让系统启用这一功能。）

    sudo mount -o rw,relatime,data=ordered,discard /dev/sdxx /mnt/newroot

接下来便可以使用rsync来迁移系统文件了，注意使用exclude选项排除不要的文件。
```
cd /mnt/oldroot  
sudo rsync -aAXHv ./ /mnt/newroot/  --exclude={dev/*,proc/*,sys/*,tmp/*,run/*,mnt/*,media/*,lost+found}  
```
如果单独分出了 /boot、/home 或者其他文件对应操作一边即可。

迁移完文件，需要生成新的 fstab 文件，所参考的文章使用了 genfstab 的命令，试了一下，Ubuntu 16.04 并不自带， apt-get 也没有，搜索一下可以下载编译安装，然而还需要安装依赖，比较麻烦，我直接对照着旧文件手动替换了一下。
```bash
#查看磁盘的uuid  
sudo blkid
sudo cp /mnt/newroot/etc/fstab /mnt/newroot/etc/fstab_bak
sudo vi /mnt/newroot/etc/fstab
```
只需要迁移的分区 uuid 替换一下就可以了。

然后在原文的安装 grub 时也遇到了问题
```bash
mount -o bind /dev /mnt/newroot/dev
mount -t proc none /mnt/newroot/proc
chroot /mnt/newroot /bin/bash

#直接执行命令结果报错  
root@ubuntu:/usr/lib# grub-install --target=x86_64-efi --recheck --debug /dev/sdc
Installing for x86_64-efi platform.
grub-install: info: cannot open '/boot/grub/device.map': No such file or directory.
grub-install: error: cannot find EFI directory.
```
查看 fstab 文件时看到旧文件 `/boot/efi was on /dev/sda2 during installation`

搜索时发现可以直接安装 boot-repair 修复。  
```
sudo add-apt-repository ppa:yannubuntu/boot-repair
sudo apt-get update
sudo apt-get install -y boot-repair
```

查看一下符合预期

![预览](https://m.nep.me/blog/p03-preview.png)

不过执行前最好提前打开一个终端窗口，因为执行的过程中可能需要执行一些命令。  
然而我打开新的终端窗口时出现了错误,然后发现之前有一个没有关闭的终端窗口。

![错误](https://m.nep.me/blog/p03-error.png)

最后执行结束的时候，结束窗口仍然说出现了错误：

> An error occurred during the repair.  
> A new file (~/Boot-Info_2016-06-04__06h04.txt) will open in your text viewer.  
> In case you still experience boot problem, indicate its content to:  
> boot.repair@gmail.com  
> You can now reboot your computer.  
> Please do not forget to make your BIOS boot on sdc (128GB) disk!  
> If your computer reboots directly into Windows, try to change the boot order in your BIOS.  
> If your BIOS does not allow to change the boot order, change the default boot entry of the Windows bootloader.  
> For example you can boot into Windows, then type the following command in an admin command prompt:  
> bcdedit /set {bootmgr} path \EFI\ubuntu\shimx64.efi  
> Boot Info Script cfd9efe + Boot-Repair extra info      [Boot-Info 26Apr2016]

然后，重启，发现并没有什么问题，已经是从 SSD 启动的了 (_ﾟ∀ﾟ_)

然后用 Grub Customizer 编辑一下启动项。  
grub 引导似乎不支持 1920x1200 的分辨率，启动时还是拉伸的图像 （
