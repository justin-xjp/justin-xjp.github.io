---
title: openwrt配置l2tp/ipsec VPN III
date: '2019-10-23 15:19'
tags:
  - linux
  - openwrt/lede
  - vpn
  - ipsec
toc: true
categories:
  - 瞎折腾
abbrlink: e7707fd9
---

[penwrt配置l2tp/ipsec VPN I](e06ca514)

[penwrt配置l2tp/ipsec VPN II](c738ae61)

[openwrt配置ss-server](2ac5ecda)

## Why

办公网络绑定了物理网卡，又没有给WIFI，确很无礼的经常在办公中使用手机微信联络工作。简单的共享热点，因为MAC的原因，手机仍然没有网络使用。层尝试用代理解决。手机设置代理，仅部分软件能够通过代理使用网络，但一些蛮横的恶霸仍然不管不顾的告诉我没有网络连接。只好尝试在本地提供VPN将网络打通。

为了不打扰工作，我在工作计算机上用VBOX搭建了openwrt。使用~~l2tp/~~ipsec，手机可以免客户端使用VPN，比较方便。手机上安装有“学习强国”，所以不打算使用SS的方法，虽然SS设置起来比较简单。

<!--more-->

本篇的目的在于设置openwrt VPN服务器作为Android 和 iPhone/iPad的网关，而无需在移动设备上额外添加软件。

## 再再出发

明确我现在想要的，xauth+ipsec PSK。根据前两轮的实操，收获了一点经验：

- LAC：相当于客户端
- LNS：相当于服务端，我应该搭建的是此设置，我的服务用IP即right
- PSA：需要生成证书并传送到终端安装，汗😓，我要是能传文件什么的，就不需要解决问题了！我要的是终端开箱即用的VPN
- strongswan：在不需要l2tp的前提下，可以完全胜任需求。所以它的设置文档值得一读。
- 需要完整的dnsmasq
- 需要添加一个独立的接口
- 需要更改的配置文件：`/etc/config/{network/firewall}`, `/etc/{firewall.user/ipsec.secrets/strongswan.conf}`, `/etc/init.d/ipsec`
- 需要证书工具

再出发前，制定本此出发的路线：

1. 梳理**hwdsl2**[^1]的自动脚本，将过程写在下面
2. 对比**openwrt配置l2tp/ipsec VPN II** 的设置，改成**openwrt**的适用脚本或方法。
3. 对照**strongswan**的[wiki文档][2]，详细注释脚本

## 梳理自动脚本

license : creative commons attribution-sharealike 3.0

需要自定义的变量：

- VPN_IPSEC_PSK=YOUR_IPSEC_PSK=''
- VPN_USER=YOUR_USERNAME=''
- VPN_PASSWORD=YOUR_PASSWORD=''

### vpnsetup.sh主文件

```shell
#!/bin/sh

export PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"

SYS_DT=$(date +%F-%T) #当前时间，$()可以将内容执行结果赋值给变量

vpnsetup "$@" #本脚本执行不需要额外参数，$@=''
exit 0

```

### vpnsetup()函数

```shell

```



## 手动流程

### 安装需求包

```
opkg update
opkg install wget dnsutils openssl iptables iproute2 gawk grep sed net-tools
```

**对于所列软件，应对比系统中是否已经存在能够替代使用的命令。查证后再继续**

### 设置变量

指定公共ip：PUBLIC_IP=?，通过手段拿到公网IP

### 安装VPN软件

list:

- libnss3-dev
- libnspr4-dev
- pkg-config 
- libpam0g-dev
- libcap-ng-dev
- libcap-ng-utils
- libselinux1-dev
- libcurl4-nss-dev
- flex
- bison
- gcc
- make
- libnss3-tools
- libevent-dev
- ppp
- xl2tpd 

### 安装SSH防护软件

- fail2ban

### 制作并安装libreswan

😓，狂吐🤮，看来我只需要安装strongswan就好了

### 编写VPN配置文件

先是关于l2tp的，跳过不看

```
XAUTH_NET="192.168.43.0/24"

XAUTH_POOL="192.168.43.10-192.168.43.250"

DNS_SRVS="8.8.8.8 4.4.4.4"

[ -n "$VPN_DNS_SRV1" ] && [ -z "$VPN_DNS_SRV2" ] && DNS_SRVS="$DNS_SRV1"
```

### /etc/ipsec.conf

```
version 2.0



config setup

  virtual-private=%v4:10.0.0.0/8,%v4:192.168.0.0/16,%v4:172.16.0.0/12,%v4:!$L2TP_NET,%v4:!$XAUTH_NET

  protostack=netkey

  interfaces=%defaultroute

  uniqueids=no



conn shared

  left=%defaultroute

  leftid=$PUBLIC_IP

  right=%any

  encapsulation=yes

  authby=secret

  pfs=no

  rekey=no

  keyingtries=5

  dpddelay=30

  dpdtimeout=120

  dpdaction=clear

  ikev2=never

  ike=aes256-sha2,aes128-sha2,aes256-sha1,aes128-sha1,aes256-sha2;modp1024,aes128-sha1;modp1024

  phase2alg=aes_gcm-null,aes128-sha1,aes256-sha1,aes256-sha2_512,aes128-sha2,aes256-sha2

  sha2-truncbug=no

conn xauth-psk

  auto=add

  leftsubnet=0.0.0.0/0

  rightaddresspool=$XAUTH_POOL

  modecfgdns=$DNS_SRVS

  leftxauthserver=yes

  rightxauthclient=yes

  leftmodecfgserver=yes

  rightmodecfgclient=yes

  modecfgpull=yes

  xauthby=file

  ike-frag=yes

  cisco-unity=yes

  also=shared
```

### /etc/ipsec.secrets

```
%any  %any  : PSK "$VPN_IPSEC_PSK"
```

### /etc/ppp/chap-secrets

似乎和l2tpd有关，查证后使用

```
"$VPN_USER" l2tpd "$VPN_PASSWORD" *
```

### 写入 /etc/ipsec.d/passwd

这个之前没有接触过，似乎是将编码后的密码写入文件供xauth使用。

```shell
VPN_PASSWORD_ENC=$(openssl passwd -1 "$VPN_PASSWORD")
conf_bk "/etc/ipsec.d/passwd" #如果存在备份文件

cat > /etc/ipsec.d/passwd <<EOF

$VPN_USER:$VPN_PASSWORD_ENC:xauth-psk

EOF
```

### /etc/sysctl.conf

每一个操作都应该备份文件。备份文件后

64位:

    SHM_MAX=68719476736
    
    SHM_ALL=4294967296
32位：

```
    SHM_MAX=4294967295

    SHM_ALL=268435456
```

文件改动：

```
# Added by hwdsl2 VPN script

kernel.msgmnb = 65536

kernel.msgmax = 65536

kernel.shmmax = $SHM_MAX

kernel.shmall = $SHM_ALL



net.ipv4.ip_forward = 1

net.ipv4.conf.all.accept_source_route = 0

net.ipv4.conf.all.accept_redirects = 0

net.ipv4.conf.all.send_redirects = 0

net.ipv4.conf.all.rp_filter = 0

net.ipv4.conf.default.accept_source_route = 0

net.ipv4.conf.default.accept_redirects = 0

net.ipv4.conf.default.send_redirects = 0

net.ipv4.conf.default.rp_filter = 0

net.ipv4.conf.$NET_IFACE.send_redirects = 0

net.ipv4.conf.$NET_IFACE.rp_filter = 0



net.core.wmem_max = 12582912

net.core.rmem_max = 12582912

net.ipv4.tcp_rmem = 10240 87380 12582912

net.ipv4.tcp_wmem = 10240 87380 12582912
```

### 更新iptables

## 后记

事情没做完，然后我放弃了，因为我做了下面这件事 [penwrt配置ss-server](openwrt-install-l2tp-ipsec-IV)
----

## 参考资料

[^1]:[IPsec VPN 服务器一键安装脚本](https://github.com/hwdsl2/setup-ipsec-vpn/blob/master/vpnsetup.sh)

[2]:https://wiki.strongswan.org/projects/strongswan/wiki	""strongswan的文档""