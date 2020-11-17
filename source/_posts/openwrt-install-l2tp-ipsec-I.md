---
title: openwrt配置l2tp/ipsec VPN I
date: '2019-10-22 11:19'
tags:
  - linux
  - openwrt/lede
  - vpn
  - ipsec
toc: true
categories:
  - 瞎折腾
abbrlink: e06ca514
---

[penwrt配置l2tp/ipsec VPN II](c738ae61)

[penwrt配置l2tp/ipsec VPN III](e7707fd9)

[openwrt配置ss-server](2ac5ecda)

## Why

办公网络绑定了物理网卡，又没有给WIFI，确很无礼的经常在办公中使用手机微信联络工作。简单的共享热点，因为MAC的原因，手机仍然没有网络使用。层尝试用代理解决。手机设置代理，仅部分软件能够通过代理使用网络，但一些蛮横的恶霸仍然不管不顾的告诉我没有网络连接。只好尝试在本地提供VPN将网络打通。

为了不打扰工作，我在工作计算机上用VBOX搭建了openwrt。使用l2tp/ipsec，手机可以免客户端使用VPN，比较方便。手机上安装有“学习强国”，所以不打算使用SS的方法，虽然SS设置起来比较简单。

<!--more-->

## 计划

通过对比几个教程，横向对比适用ubuntu的自动安装脚本，再自行决定调整过程。

## 安装 strongswan xl2tpd

运行`opkg install strongswan xl2tpd`

## /etc/ipsec.conf配置[^1] [^6] [^7]
```
config setup
	


conn %default
        ikelifetime=60m
        keylife=20m
        rekeymargin=3m
        keyingtries=1
        #keyexchange=ikev1       #[^1]
    	#authby=secret
    	#ike=aes128-sha1-modp1024,3des-sha1-modp1024! 
    	#esp=aes128-sha1-modp1024,3des-sha1-modp1024! 

conn l2tp
        keyexchange=ikev1 # IKE版本
        left=%defaultroute #应设置外网可以访问的ip     [^6]
        #leftsubnet=0.0.0.0/0 #left是服务器IP，被手机连接的ip。
        leftprotoport=17/1701
        authby=secret
        leftfirewall=no
        right=%any
        rightprotoport=17/%any
        type=transport
        auto=add
```

## /etc/ipsec.secrets配置，可新建[^1][^6][^7]

```
# ipsec.secrets - strongSwan IPsec secrets file
xxx.xxx.xxx.xxx %any: PSK "<PSK>"#PSK可以是自行设定的字符串
```
PSK即是**预共享密钥**

## l2tp

**/etc/xl2tpd/xl2tpd.conf** 的设置，这里有两个概念很扰人"LAC"和"lns"，~~按协议简析  [^8]所述理解，应该配置LAC：~~，LAC即是用在路由或交换机中的cilent，所以放弃[^1]中的配置做法。

```
[lns default]
ip range = 192.168.137.2-192.168.137.200
local ip = 10.10.0.1
require chap = yes
refuse pap = yes
require authentication = yes
name = LinuxVPNserver
ppp debug = yes
pppoptfile = /etc/ppp/options.xl2tpd
length bit = yes
bps = 1000000
```



## PPP，chap验证

**/etc/ppp/options.xl2tpd**[^6][^7] 


```
require-mschap-v2
#asyncmap 0
#auth

#hide-password
#modem
########
debug
name l2tpd
lcp-echo-interval 30
lcp-echo-failure 4
ms-dns  8.8.8.8
ms-dns  8.8.4.4
noccp
auth
crtscts #都有
idle 600
mtu 1200
mru 1200
nodefaultroute
debug
lock
proxyarp
connect-delay 2500

```

连接密码 chap-secrets, **/etc/ppp/chap-secrets**

```
# Secrets for authentication using CHAP
# client        server  secret                  IP addresses
<user>          *       <password>              *
```

有两个星号，第一个表示以后所有使用PPP作为用户认证的服务，都可以使用这个用户名和密码，包括PPTP和L2TP都可以使用loginname。第二个星号表示这个用户可以从任何IP登录。如果你希望控制一下，可以把星号改成具体的值来限制。[^7]

## ip转发设置

**/etc/sysctl.conf**



```
net.ipv4.conf.all.rp_filter = 0
net.ipv4.conf.default.rp_filter = 0
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0
```

或是

```
net.ipv4.ip_forward = 1 #default 文件中又，可不用
net.ipv4.conf.default.rp_filter = 0 #same
net.ipv4.conf.default.accept_source_route = 0
net.ipv4.conf.all.send_redirects = 0 #same
net.ipv4.conf.default.send_redirects = 0 #same
net.ipv4.conf.all.log_martians = 0
net.ipv4.conf.default.log_martians = 0 
net.ipv4.conf.all.accept_redirects = 0 #same
net.ipv4.conf.default.accept_redirects = 0 #same
net.ipv4.icmp_ignore_bogus_error_responses = 1
```




接下来是端口开放,500,1701,4500

```
iptables -t nat -A POSTROUTING -s 10.10.0.0/24 -o eth0 -j MASQUERADE
```

或者

```
iptables -A INPUT -m policy --dir in --pol ipsec -j ACCEPT
iptables -A FORWARD -m policy --dir in --pol ipsec -j ACCEPT
iptables -t nat -A POSTROUTING -m policy --dir out --pol none -j MASQUERADE
iptables -A FORWARD -i ppp+ -p all -m state --state NEW,ESTABLISHED,RELATED -j ACCEPT
iptables -A FORWARD -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A INPUT -m policy --dir in --pol ipsec -p udp --dport 1701 -j ACCEPT
iptables -A INPUT -p udp --dport 500 -j ACCEPT
iptables -A INPUT -p udp --dport 4500 -j ACCEPT
iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o eth0 -j MASQUERADE

service iptables save
service iptables restart
```



## 目前状态

通通不可用，vbox用镜像回到了折腾前状态。

----

## 参考资料：

[^1]: https://blog.csdn.net/ypbsyy/article/details/81325906
[^2]: IPsec VPN 服务器一键安装脚本 https://github.com/hwdsl2/setup-ipsec-vpn/blob/master/README-zh.md
[^3]:另一个一键脚本：https://github.com/teddysun/across/blob/master/l2tp.sh
[^4]:openwrt的说明：https://openwrt.org/docs/guide-user/services/vpn/ipsec/strongswan/basics
[^5]:PSec/IKEv2 VPN安装脚本 For CentOS/Debian/Ubuntu https://qiaodahai.com/ipsec-ikev2-installation-script-for-centos-debian-ubuntu.html
[^6]: [L2TP/IPSEC搭建VPN详细教程](https://segmentfault.com/a/1190000006125737) https://segmentfault.com/a/1190000006125737
[^7]:[在 CentOS 7 上部署 L2TP/IPSec VPN 服务](https://www.robberphex.com/centos-7-l2tp-ipsec-vpn/)
[^8]: https://www.cnblogs.com/Shepherdzhao/p/8639794.html

