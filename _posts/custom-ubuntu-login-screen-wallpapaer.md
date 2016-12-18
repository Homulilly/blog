---
title: 自定义 Ubuntu 登录界面壁纸
date: 2016-08-02 13:20:30
tags:
 - Ubuntu
---

现在用的两个显示器，分辨率不一样、屏幕颜色也不一样，一个是IPS、一个是TN，为了避免精神分裂，所以壁纸还是需要设为不一样的图片，然而Ubuntu似乎比较困难，Win这方面是真好。  
只好把两张图片合体了，然后设置为适合宽度的方式，话说 Ubuntu 的平铺壁纸逻辑我是真的无法理解。然而这样登录界面又坑了，两个屏幕的壁纸是镜像的，而且都是一半一半。  
只好想办法单独设置了。  

一开始我直接去 dconf 修改或者使用 gsettings 命令都没有效果（ 搜索了一下，发现需要下面的操作步骤：  

```
sudo -i
xhost +SI:localuser:lightdm
su lightdm -s /bin/bash
gsettings set com.canonical.unity-greeter draw-user-backgrounds 'false'
gsettings set com.canonical.unity-greeter background 'path-to-image'
exit
```

如果想去除界面上的白点：  
```
gsettings set com.canonical.unity-greeter draw-grid 'false'
```

虽然这样切回单屏幕时壁纸还是会傻逼，至少会好一些了。  

系统：Ubuntu 16.04 LTS  
参考：https://askubuntu.com/questions/455849/unity-greeter-does-not-display-custom-wallpaper  
