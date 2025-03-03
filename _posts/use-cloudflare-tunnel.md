---
title: 使用 Cloudflare Tunnel 搭建内网穿透
date: 2024-11-13 22:01:53
tags:
 - Cloudflare
 - Tunnel
categoriez:
 - Note
---

不得不说，Cloudflare 真是赛博菩萨，提供的服务又多又好，还可以免费用。  

最近需要使用内网穿透，试一试 Cloudflare 的 [Tunnel](https://www.cloudflare.com/zh-cn/products/tunnel/) 。

```
用户 <---> Cloudflare <--> Cloudflare Tunnel <--> 源服务器(可以位于内网)
```

> 使用感受：适合代理 web 服务，客户端无需安装程序，如果代理 tcp 服务比如 ssh 需要在客户机安装 cloudflared ，无法直接转发。 
> 如果有在公网的 VPS ，推荐使用 [frp](https://github.com/fatedier/frp)
<!--more-->

## 准备

- 需要有一个 Cloudflare 账户
- 需要开通 **Zero Trust** : 访问 [dash.cloudflare.com](https://dash.cloudflare.com/)，点击左侧的 **Zero Trust**
- 选择免费套餐
    - 虽然账单是 0 ，但是还要走一遍支付流程

## 安装 Cloudflared

Cloudflared 是 Cloudflare 维护的开源项目： [Github](https://github.com/cloudflare/cloudflared) 

Linux 可以直接从 [Github - Releases](https://github.com/cloudflare/cloudflared/releases) 下载安装文件

```sh
curl -L 'https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64' -o /usr/bin/cloudflared

chmod +x /usr/bin/cloudflared
```

## 登陆
```sh
root@Debian:/~# cloudflared tunnel login
Please open the following URL and log in with your Cloudflare account:

https://dash.cloudflare.com/argotunnel?aud=&callback=https%3A%2F%2Flogin.cloudflareaccess.org%2************************

Leave cloudflared running to download the cert automatically.
```

在浏览器内访问链接，选择一个项目进行授权。  

## 创建 Tunnel
可以通过网页进行创建
- 访问 Zero Trust 的面板 [https://one.dash.cloudflare.com/](https://one.dash.cloudflare.com/)
- 选择账户
- 点击左侧菜单的 **Network** > **Tunnel** 
- 点击按钮 **Add a tunnel**
- 选择 **Cloudflared**
- `Name your tunnel`:  设置一个名字
- `Install and run a connector`: 选择系统后，已经提供了对应的命令
```
sudo cloudflared service install AAABBBCCC....
```

然后进入转发设置页面，比如如图是将 `mysubdomain.nep.me` 转发到 `http://192.168.1.10:8888`

![cf-tunnel-config.png](https://m.nep.me/blog/post/cf-tunnel-config.png)

点击保存即可，查看 `Status` 正常应该显示为 `HEALTHY` 。  

然后就可以访问设置的域名查看。  

## 设置访问限制

服务暴漏在公网要考虑到安全问题，如果源网站没有安全限制措施，可以使用 Cloudlflare 提供的 **One-time PIN** ，这是默认添加的，可以在 **Settings** > **Authentication** > **Login methods** 中查看。

然后访问左侧菜单 **Access** > **Applications** > **Add an application**

选择 Self-hosted ，第一步是 **Configure application**

- `Application name` 设置名称
- `Session Duration` 有效持续时间
- `Application domain` 需要保护的域名与路径，比如上面设置的 `mysubdomain.nep.me`

其余保持默认即可。 

在下一步中，需要设置 Policy，这是用来控制谁可以访问你的应用：

1. 基本设置：
    - `Policy name`: 为策略设置一个名称
    - `Action`: 选择 "Allow" 允许符合条件的访问

2. Configure rules:
    - 可以设置多种规则组合
    - 常用规则包括：
      - Email 域名限制
      - IP 地址范围
      - 地理位置

3. 认证方式：
    - 默认的 One-time PIN 方式最简单
    - 访问时需要输入邮箱
    - Cloudflare 会发送一次性验证码
    - 验证通过后即可访问

完成设置后，当访问相关应用后，会先经过 Cloudflare 的身份验证。

不过这样有些略显麻烦，相较于直接将服务暴露在公网中，还是习惯转发一个代理回到内网使用相应的服务。  

## 卸载 cloudflared 服务

```sh
cloudflared service uninstall
dpkg --remove cloudflared
```