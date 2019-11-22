---
title: penwrt配置ss-server
date: '2019-10-24 12:19'
tags:
  - linux
  - openwrt/lede
  - vpn
  - ipsec
toc: true
categories:
  - 瞎折腾
abbrlink: 2ac5ecda
---

## Why

办公网络绑定了物理网卡，又没有给WIFI，确很无礼的经常在办公中使用手机微信联络工作。简单的共享热点，因为MAC的原因，手机仍然没有网络使用。层尝试用代理解决。手机设置代理，仅部分软件能够通过代理使用网络，但一些蛮横的恶霸仍然不管不顾的告诉我没有网络连接。只好尝试在本地提供VPN将网络打通。

为了不打扰工作，我在工作计算机上用VBOX搭建了openwrt。使用~~l2tp/~~ipsec，手机可以免客户端使用VPN，比较方便。不再坚持手机端的便利。



本篇的目的在于设置openwrt VPN服务器作为Android 和 iPhone/iPad的网关，

## 安装软件

```
opkg update
opkg install shadowsocks-libv-server luci-app-shadowsocks-libev
```

## 配置服务器

### 打开端口

查看后需打开9001

### 服务器配置

service》shadowsocks-libv>  ss-server

设置密码，将端口绑定到接收用网址。

应用，然后OK。

