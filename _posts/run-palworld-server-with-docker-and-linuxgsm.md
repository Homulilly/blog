---
title: 使用 Docker 与 LinuxGSM 运行 Palworld 服务器
date: 2024-02-07 21:16:30
tags:
 - Docker
 - LinuxGSM
 - Palworld
categories:
  - Note
---
我也买了一份帕鲁，掏出了我的零刻小主机开了个自己玩的服务器。
用 Docker 是非常的简单，不过今天起床发现游戏更新了，但是 Docker 还没有更新，我进不去了，不得已用 LinuxGSM 重新搭了一个过度
两个都记录一下。

<!-- more -->

## Docker 版本
[使用介绍](https://github.com/thijsvanloef/palworld-server-docker/blob/main/docs/zh-CN/README.md) 

创建文件夹与 `docker-compose.yml` 文件，我是放在用户目录下的 palworld 文件夹内
```bash 
mkdir palworld && cd palworld

vim docker-compose.yml
```

填入下面的内容  
```yml
services:
  palworld:
    image: thijsvanloef/palworld-server-docker:latest
    restart: unless-stopped
    container_name: palworld-server
    ports:
      - 8211:8211/udp
      - 27015:27015/udp
    environment:
      - PUID=1000
      - PGID=1000
      - PORT=8211 # 可选但推荐
      - PLAYERS=16 # 可选但推荐
      - SERVER_PASSWORD=worldofpals # 可选但推荐
      - MULTITHREADING=true
      - RCON_ENABLED=true
      - RCON_PORT=25575
      - TZ=Asia/Shanghai
      - ADMIN_PASSWORD=adminPasswordHere
      - COMMUNITY=false  # 如果您希望服务器显示在社区服务器页中，请启用此选项（注意配置SERVER_PASSWORD!）
      - SERVER_NAME=World of Pals
      - SERVER_DESCRIPTION=palworld-server-docker by Thijs van Loef
    volumes:
      - ./palworld:/palworld/
```

然后执行 `docker compose up -d` 就可以开启服务器，编辑世界设置可以直接在 `docker-compose.yml` 填写，或是编辑目录中的 `palworld/Pal/Saved/Config/LinuxServer/PalWorldSettings.ini` 文件

使用 Docker 版本比较方便的备份，备份存档的话是备份 `palworld/Pal/Saved/` 文件夹 

```bash
docker exec palworld-server backup
```

可以直接在 `docker-compose.yml` 中自定义服务器配置，比如我设置无死亡掉、10 倍经验、孵蛋 0 耗时、关闭建筑风华，需要在 `environment:` 部分加入下面的内容 
```yaml
      - DEATH_PENALTY=None
      - EXP_RATE=10.0
      - PAL_EGG_DEFAULT_HATCHING_TIME=0
      - BUILD_OBJECT_DETERIORATION_DAMAGE_RATE=0
```

## LinuxGSM 版本

[LinuxGSM](https://linuxgsm.com/) 是管理服务器脚本的合集，支持安装、更新、管理游戏服务器。用来管理 Palworld 的服务器也是非常的简单。  
参考 [https://linuxgsm.com/servers/pwserver/](https://linuxgsm.com/servers/pwserver/) 按照步骤进行即可  

LinuxGSM 不支持 root 权限运行，比较推荐创建一个单独的用户来运行。   

### 安装 LinuxGSM
首先安装依赖，我使用的是 Debian 12 
```bash
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install curl wget file tar bzip2 gzip unzip bsdmainutils python3 util-linux ca-certificates binutils bc jq tmux lib32gcc-s1 lib32stdc++6 netcat-openbsd
```

安装 LinuxGSM
```bash
mkdir palworld && cd palworld

curl -Lo linuxgsm.sh https://linuxgsm.sh && chmod +x linuxgsm.sh && bash linuxgsm.sh pwserver  
```

安装 Palworld 的脚本
```bash
./pwserver install
```

然后执行
```bash
./pwserver start
```
即可运行 Palworld 服务器

### 游戏存档

服务器的存档位于 `~/palworld/serverfiles/Pal/Saved/` 文件夹内

世界设置位于
``` 
~/palworld/serverfiles/Pal/Saved/Config/LinuxServer/PalWorldSettings.ini
```

可以使用网站生成服务器配置
https://najoast.github.io/PalWorldSettingsUI/

## 其他工具

- [帕鲁配种查询](https://palworld.caimogu.cc/breed.html)
- [幻兽帕鲁服务器管理工具](https://github.com/zaigie/palworld-server-tool)

## 一些问题
重启服务器后要在白天到据点激活一下帕鲁们，不然全都偷懒的饭都不吃了 
