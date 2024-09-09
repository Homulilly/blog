---
title: 黑群晖
date: 2023-11-18 18:03:34
tags: 
 - DSM
---

现在黑群晖的安装异常的简单了，记录一下。

- 安装方式 ：PVE 8
- DSM 7.2

<!--more-->
## 创建虚拟机
### 首先创建一个 `VM`

- 系统选择不使用镜像
- 硬盘不要添加(在添加界面可以删除)
- 记录 `vmid`

### 导入引导镜像
从 [Gihub](https://github.com/wjz304/rr/releases) 下载引导镜像，解压得到 `arpl.img` 上传到 PVE 的镜像存储池中。    
记录镜像文件的路径 `/var/lib/vz/template/iso/arpl.img`  

然后将镜像导入虚拟机：
```
qm importdisk $vmid /var/lib/vz/template/iso/arpl.img local
```
如果 PVE 没有删除 local-lvm，上面最后 local 替换为 local-lvm 
然后前往 PVE 的 Web 管理 ，可以看到有一个未使用磁盘，选中 > 编辑，总线选择 SATA 。

### 直通硬盘
需要给群晖添加额外的磁盘用于存储，我选择的是直通一个 SATA 硬盘。

1. 直通单块硬盘：   
    查询硬盘 ID 
    ```
    ls -l /dev/disk/by-id/
    ```
    使用下面的命令直通 SATA 硬盘
    ```
    qm set $vmid -sata1 /dev/disk/by-id/$disk-id
    ```
    使用这种方法在群晖里显示的是 `QEMU DISK`，无法查看 S.M.A.R.T. 信息，不过 7.2 系统看着已经不显示 S.M.A.R.T. 的详细信息了。

2. 直通 PCI SATA 控制器  
    查看 S.M.A.R.T. 信息可以直通 PCI 设备，需要 PVE 的自身系统不能安装在该 PCI 下面。  
    方法更简单，直接在 PVE 网页管理界面就可以操作。
 
    首先通过 `lspci` 查看信息，记录要直通的 SATA 控制器的编号。    
    然后在虚拟机 WEB 管理界面，选择硬件，点击 `添加` > `PCI 设备`，在设备中选择上一步中查看的编号。  

    选择 `Raw Device` ，再勾选 `高级` > `PCI-Express` 即可。  

    >我是先尝试了直通单块硬盘，黑群晖关机后，先分离硬盘，然后重新添加了 PCI 设备，直接再启动黑群晖即可，PVE 也没有重启就生效了。
    也不需要重新安装 DSM 系统。  

    进入黑群晖系统，在存储管理中可以查看到硬盘显示为 SSD，并且状况信息中可以检测 S.M.A.R.T. 信息了。

### 编译引导
在 PVE 管理界面，选择选项，引导顺序，编辑选择引导镜像导入的磁盘，然后选择启动。

按照界面提示 使用浏览器访问 http://IP:7681/ ，依次

- `Choose the model`，选择要引导的型号
- `Choose a Build Numble`，选择系统版本
- `Build the loader`，构建引导
- `Boot the loader`，启动

等待一段时间后访问 http://IP ，可以进入安装向导，DSM 的更新选择 手动安装 。

### 问题
#### 移除访问 DSM 主页的端口 5001
- 已配置域名访问  

可以通过设置反向代理实现  
打开 `控制面板` > `登录门户` > `高级` > `反向代理服务器` ，选择 `新增`  
设置 443 端口指向 localhost 的 5001  

<img src="https://m.nep.me/blog/post/dsm-reverse-proxy.jpg" alt="dsm-reverse-proxy" width=50% />

#### 视频缩略图索引
- 需要安装第三方的 FFmpeg 替换官方

1. 添加第三方源 
    ```
    https://spk7.imnks.com/
    ```

2. 在套件中心安装 `FFmpeg 6`  
3. 替换  
ssh 登陆群晖，执行下面的命令
    ```bash
    sudo -i
    # 备份
    mv /usr/bin/ffmpeg /usr/bin/ffmpeg.bak
    # 替换
    ln -s /var/packages/ffmpeg6/target/bin/ffmpeg /usr/bin/ffmpeg
    ```
4. 设置文件夹权限  
    `控制面板` > `共享文件夹` > 编辑需要索引的文件夹权限  
    选择 `系统内部用户账户`，将 `sc-ffmpeg6` 的权限设置为 `可读写`  

5. 重建索引  
    前往 Photos 程序以及 `控制面板 > 索引服务` 点击重建索引  
    生成速度比较慢。  
    索引完成后，在 DSM 主页可以查看视频缩略图进度  
    ![dsm-video-thumb](https://m.nep.me/minio/d/blog/post/dsm-video-thumb.jpg)

### 注意事项
#### 搬迁网络时，取消静态 IP 
如果需要更换网络环境，记得先在群晖的设置中取消静态 IP 的设置。

引导是通过 DHCP 获取的 IP ，如果群晖内设置了静态 IP ，引导完就访问不了啦，这种情况下，可以先将主路由器的 IP 网段设置与原网络一致，然后就能访问了。 

### 参考
- [PVE安装黑群晖DSM7虚拟机及直通设置](https://blog.mstg.top/archives/762)  
- [黑群晖NAS (ARPL引导)安装教程](https://post.smzdm.com/p/a4p69v7k/)
- [群晖DSM7.0-7.1.1 Synology Photos 视频缩略图处理](https://note.youdao.com/ynoteshare/index.html?id=6ec33870f01f5ab65465c6675722b1a9)