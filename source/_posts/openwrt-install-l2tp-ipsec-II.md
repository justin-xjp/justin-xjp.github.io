---
title: openwrt配置l2tp/ipsec VPN II
date: '2019-10-22 15:19'
tags:
  - linux
  - openwrt/lede
  - vpn
  - ipsec
toc: true
categories:
  - 瞎折腾
abbrlink: c738ae61
---



写在看之前：根据我血的教训，看前应该明确自己的需求，你需要的是IKEv2的ipsec连接吗？不是的话，不用往下看了……

[penwrt配置l2tp/ipsec VPN I](e06ca514)

## Why

办公网络绑定了物理网卡，又没有给WIFI，确很无礼的经常在办公中使用手机微信联络工作。简单的共享热点，因为MAC的原因，手机仍然没有网络使用。层尝试用代理解决。手机设置代理，仅部分软件能够通过代理使用网络，但一些蛮横的恶霸仍然不管不顾的告诉我没有网络连接。只好尝试在本地提供VPN将网络打通。

为了不打扰工作，我在工作计算机上用VBOX搭建了openwrt。使用~~l2tp/~~ipsec，手机可以免客户端使用VPN，比较方便。手机上安装有“学习强国”，所以不打算使用SS的方法，虽然SS设置起来比较简单。



本篇的目的在于设置openwrt VPN服务器作为Android 和 iPhone/iPad的网关，而无需在移动设备上额外添加软件。

## 再出发

~~本次参照资料：How to set up an OpenWRT router/gateway as an IPsec/L2TP gateway for Andoid and iPhone clients[^1]   ————~~文章过于古董，是10.03以前的openwrt。还能用吗？

理论上，连接仍被分为了两部分：

1. IPsec负责外部的连接，通过PSK或是X.509证书验证。
2. L2TP连通内部PPP通道，通过 user/password验证，通常适用MSCHAP-v2轮询模式

再次信任并尝试按照[官方的手册][6]搭建vpn server。

> This is a tested example which should allow anyone to easily setup a secure and working VPN server. 

### 环境假设

lan 是 **192.168.0.0**/16	根据实际情况，我需要改为 192.168.0.0/16

router's IP **192.168.0.1**  根据实际情况，（其实我不知道什么是实际情况，此ip是vpn的服务接收IP，还是什么？）应该是本机的服务IP吧，猜 **192.168.56.2**

/etc/config/network:

```
config interface 'lan'
	option type		'bridge'
	option ifname		'eth0'
	option proto		'static'
	option ipaddr		'192.168.0.1'#改为自身设置
	option netmask		'255.255.0.0'
	option ip6assign	'60'
```

yes, 我猜对了，但我需要对**netmask**进行更改，以适应此设置。

文档假设WAN是可以变化的，所以在设置中并没有提及。并假设 VPN的服务名为：**VpnTest**，国家是Finland(FI)，记得自行调整。当有domain域名时，应设置指向`vpntest.lan`，而静态WAN ip，应查看`mk_server.sh`。（这两句完全不理解什么意思，与我无关的设置应尽少提及）

`vpntest.lan` 由以下文件中产生：

	1. `ipsec.conf`中的**leftid**
 	2. `mk_server.sh` 中的 **SRVNAME** 和 **IPADDR** 变量

可以在脚本中修改**ORG**（组织名称）。请根据自身 `lan` 情况更改:

	1. `ipsec.conf`: **leftsubnet , rightsubnet , rightdns**
 	2. `strongswan.conf` : **charon.dns1, charon.plugins.dhcp.server**

教程中的其他设置可以直接拷贝来使用，整个教程通过了 Mac OS X 和 iPhone的测试，Android和Windows应该更没有问题。

### 动态IP见原文

## 安装Packages

安装需要软件：

```
opkg update
opkg install curl strongswan-default strongswan-pki ipset strongswan-mod-openssl strongswan-mod-curl strongswan-mod-dhcp strongswan-mod-eap-tls strongswan-mod-eap-identity strongswan-mod-kernel-libipsec kmod-tun openssl-util strongswan-mod-test-vectors strongswan-mod-farp
```

另外，你需要完整的dnsmasq使DHCP工作：

```
/etc/init.d/dnsmasq stop
opkg remove dnsmasq
opkg install dnsmasq-full
/etc/init.d/dnsmasq start
```

## 配置文件

### /etc/config/network

定义 ipsec 的接口

```
config interface 'ipsec'
	option ifname		'ipsec0'
	option proto		'none'
	option defaultroute	'0'
	option peerdns		'0'
	option ipv6		'0'
```

### /etc/firewall.user

添加转发规则

```
iptables -I INPUT   -m policy --dir in  --pol ipsec --proto esp -j ACCEPT
iptables -I FORWARD -m policy --dir in  --pol ipsec --proto esp -j ACCEPT
iptables -I FORWARD -m policy --dir out --pol ipsec --proto esp -j ACCEPT
iptables -I OUTPUT  -m policy --dir out --pol ipsec --proto esp -j ACCEPT
```

### /etc/config/firewall

新建 vpn zone

```
config zone
	option name		'vpn'
	list network		'ipsec'
	option input		'ACCEPT'
	option output		'ACCEPT'
	option forward		'ACCEPT'
	option masq		'1'
	option mtu_fix		'1'
```

允许转发

```
config forwarding
	option src		'vpn'
	option dest		'lan'

config forwarding
	option src		'lan'
	option dest		'vpn'

config forwarding
	option src		'vpn'
	option dest		'wan'
```

添加规则，允许IPSec连接 穿过防火墙

```
config rule
	option name		'IPSec-ESP'
	option src		'wan'
	option proto		'esp'
	option target		'ACCEPT'

config rule
	option name		'IPSec-IKE'
	option src		'wan'
	option dest_port	'500'
	option proto		'udp'
	option target		'ACCEPT'

config rule
	option name		'IPSec-NAT-T'
	option src		'wan'
	option dest_port	'4500'
	option proto		'udp'
	option target		'ACCEPT'

config rule
	option name		'IPSec-Auth-Header'
	option src		'wan'
	option proto		'ah'
	option target		'ACCEPT'
```

可选选项：允许igmp pinging:

```
config rule
	option name		'Allow-Ping-From-VPN'
	option src		'vpn'
	option proto		'igmp'
	option icmp_type	'echo-request'
	option family		'ipv4'
	option target		'ACCEPT'

config rule
	option name		'Allow-Ping-To-VPN'
	option dest		'vpn'
	option proto		'igmp'
	option icmp_type	'echo-request'
	option family		'ipv4'
	option target		'ACCEPT'
```

### /etc/init.d/ipsec

替换文件中的内容为以下简单脚本（此文件适用ipsec/strongswan文件进行配置），注意文件权限

```
#!/bin/sh /etc/rc.common
# ipsec init script

START=90
STOP=10

USE_PROCD=1
PROG=/usr/lib/ipsec/starter

service_running() {
	ipsec status > /dev/null 2>&1
}

reload_service() {
	running && {
		ipsec rereadall
		ipsec reload
		return
	}

	start
}

check_ipsec_interface() {
	procd_add_interface_trigger "interface.*" "ipsec0" /etc/init.d/ipsec  reload
}

service_triggers() {
	procd_add_reload_trigger "ipsec"
	check_ipsec_interface
}

start_service() {
	procd_open_instance
	procd_set_param command $PROG --daemon charon --nofork
	procd_set_param respawn
	procd_close_instance
}
```

新建的文件需要`chmod 755 ipsec`

### /etc/ipsec.secrets

```
      # /etc/ipsec.secrets - strongSwan IPsec secrets file
      : RSA host-vpn.der
```

RSA？ 一直简单使用的RSK，看来要改了，关键是 `host-vpn.der`在哪？怎么来？

### /etc/ipsec.conf

```
# ipsec.conf - strongSwan IPsec configuration file

# basic configuration

config setup
	strictcrlpolicy = no
	uniqueids = no

# Add connections here.

conn default
	keyexchange = ikev2
	ike = aes128-sha1-modp1024,aes128-sha1-modp1536,aes128-sha1-modp2048,aes128-sha256-ecp256,aes128-sha256-modp1024,aes128-sha256-modp1536,aes128-sha256-modp2048,aes256-aes128-sha256-sha1-modp2048-modp4096-modp1024,aes256-sha1-modp1024,aes256-sha256-modp1024,aes256-sha256-modp1536,aes256-sha256-modp2048,aes256-sha256-modp4096,aes256-sha384-ecp384,aes256-sha384-modp1024,aes256-sha384-modp1536,aes256-sha384-modp2048,aes256-sha384-modp4096,aes256gcm16-aes256gcm12-aes128gcm16-aes128gcm12-sha256-sha1-modp2048-modp4096-modp1024,3des-sha1-modp1024!
	esp = aes128-aes256-sha1-sha256-modp2048-modp4096-modp1024,aes128-sha1,aes128-sha1-modp1024,aes128-sha1-modp1536,aes128-sha1-modp2048,aes128-sha256,aes128-sha256-ecp256,aes128-sha256-modp1024,aes128-sha256-modp1536,aes128-sha256-modp2048,aes128gcm12-aes128gcm16-aes256gcm12-aes256gcm16-modp2048-modp4096-modp1024,aes128gcm16,aes128gcm16-ecp256,aes256-sha1,aes256-sha256,aes256-sha256-modp1024,aes256-sha256-modp1536,aes256-sha256-modp2048,aes256-sha256-modp4096,aes256-sha384,aes256-sha384-ecp384,aes256-sha384-modp1024,aes256-sha384-modp1536,aes256-sha384-modp2048,aes256-sha384-modp4096,aes256gcm16,aes256gcm16-ecp384,3des-sha1!
	mobike = yes        
	dpdaction = clear
	dpddelay = 60s
	left = %any
	#leftid = <VPN SERVER ID / DOMAINNAME!!!>
	leftid = vpntest.lan
	leftsubnet = 0.0.0.0/0
	leftcert = host-vpn.der
	leftsendcert = always
	right = %any
	rightauth = eap-tls
	rightsourceip = %dhcp
	#rightdns = <LAN DNS SERVER>
	rightdns = 192.168.0.1 #lan.ip 
	eap_identity = %identity
	forceencaps = yes
	auto = add
```

### /etc/strongswan.conf

```
# strongswan.conf - strongSwan configuration file
#
# Refer to the strongswan.conf(5) manpage for details
#
# Configuration changes should be made in the included files

charon {
	dns1 = 192.168.0.1 #lan.ip
	load_modular = yes
	plugins {
		include strongswan.d/charon/*.conf
		dhcp {
			force_server_address = yes
                        #use_server_port = yes
                        # uncomment the line above if log shows that DHCP 
                        # offer can't be accepted  
			identity_lease = yes
			server = 192.168.255.255
			# use LAN broadcast address here, not IP address.        
		}
	}
}

pluto {
	threads = 8
	dns1 = 192.168.0.1 #lan.ip
}
      
libstrongswan {
	# set to n, the DH exponent size is optimized
	#dh_exponent_ansi_x9_42 = no
	#crypto_test {
	#	on_add = yes
	#}
}

include strongswan.d/*.conf
```

## 证书工具

以下几个脚本提供了证书管理功能：

	1. clean.sh : 清楚所有生成的证书
 	2. mk_server.sh : 生成根证书和服务器证书，只执行一次
 	3. mk_user.sh: 生成客户端证书

当生成完服务器根证书后，应立即去除 mk_server.sh，如再次生成根证书，原来生成的用户证书将不再匹配。此外，对这些脚本的更改还允许更改证书的有效期。

### /etc/ipsec.d/clean.sh

```
#!/bin/sh
rm /etc/ipsec.d/*/*.* 2> /dev/null
```

### /etc/ipsec.d/mk_server.sh

```
#!/bin/sh

SRVNAME="vpntest.lan"
IPADDR="vpntest.lan"
#Uncomment only if vpn server is behind a static IP
#IPADDR=$(. /lib/functions/network.sh; network_get_ipaddr ip wan; echo $ip)

COUNTRY="FI"
ORG="VpnTest"
#Change above to your org and country code

VALIDDAYS="3650"
LIFETIME="730"

ipsec pki --gen --type rsa --size 4096 --outform der > private/strongswan.der
chmod 600 private/strongswan.der

ipsec pki --self --ca --lifetime $VALIDDAYS --in private/strongswan.der --type rsa --dn "C=$COUNTRY, O=$ORG, CN=$ORG Root CA" --outform der > cacerts/strongswan.der
openssl x509 -inform DER -in cacerts/strongswan.der -out cacerts/strongswan.pem -outform PEM

ipsec pki --print --in cacerts/strongswan.der

ipsec pki --gen --type rsa --size 4096 --outform der > private/host-vpn.der
chmod 600 private/host-vpn.der

ipsec pki --pub --in private/host-vpn.der --type rsa | ipsec pki --issue --lifetime $LIFETIME --cacert cacerts/strongswan.der --cakey private/strongswan.der --dn "C=$COUNTRY, O=$ORG, CN=$SRVNAME" --san=$SRVNAME --san $IPADDR --san @$IPADDR --flag serverAuth --flag ikeIntermediate --outform der > certs/host-vpn.der
ipsec pki --print --in certs/host-vpn.der
```

### /etc/ipsec.d/mk_user.sh

```
#!/bin/sh

COUNTRY="FI"
ORG="VpnTest"

LIFETIME="730"

echo "Enter userid (no spaces or special characters):"
read USERID

[ -z "$USERID" ] && {
	echo Empty user ID. Cancelling.
	exit 0
}

echo "Enter fullname:"
read NAME

[ -z "$NAME" ] && {
	echo Empty full name. Cancelling.
	exit 0
}

mkdir -p /etc/ipsec.d/p12 2> /dev/null

ipsec pki --gen --type rsa --size 2048 --outform der > private/$USERID.der
chmod 600 private/$USERID.der

ipsec pki --pub --in private/$USERID.der --type rsa | ipsec pki --issue --lifetime $LIFETIME --cacert cacerts/strongswan.der --cakey private/strongswan.der --dn "C=$COUNTRY, O=$ORG, CN=$USERID" --san "$USERID" --outform der > certs/$USERID.der
openssl rsa -inform DER -in private/$USERID.der -out private/$USERID.pem -outform PEM

openssl x509 -inform DER -in certs/$USERID.der -out certs/$USERID.pem -outform PEM
openssl pkcs12 -export -inkey private/$USERID.pem -in certs/$USERID.pem -name "$NAME's VPN Certificate" -certfile cacerts/strongswan.pem -caname "$ORG Root CA" -out p12/$USERID.p12
```

## 客户端配置

用client.sh生成客户证书需要两个前提：

1. cacerts/strongswan.pem
2. p12/USERID.p12

将上述两个文件送到终端机上（手机），先发送strongswan.pem，并安装，然后发送USERID.p12，并安装。

`SRVNAME`是在mk_server.sh中设置的，默认是`vpntest.lan`，USERID是运行`mk_client.sh`生成的。

复制证书文件道客户端（电邮发送也行），并安装，定义连接参数：

```
VPN Type:		IKEv2
Server Address:		server ip address or url
Remote ID:		SRVNAME
Local ID:		USERID

Authentication settings:
	Method:		Certificate
	Certificate:	USERID.p12
```



## 后记

结束了？？？

使用的 IKEv2，手机上仍需要客户端来实现，不是很好的感受。

我要换xauth

## 总结：

做事情前一定要先概览一下全文，是不是自己所需。再做决定做不做。这篇权当原文的大概翻译，开始新的冒险吧。

## 参考资料：

[^1]:[How to set up an OpenWRT router/gateway as an IPsec/L2TP gateway for Andoid and iPhone clients](https://www.mayrhofer.eu.org/post/l2tp-ipsec-gateway-for-mobile-phones/)
[^2]: IPsec VPN 服务器一键安装脚本 https://github.com/hwdsl2/setup-ipsec-vpn/blob/master/README-zh.md
[^3]:另一个一键脚本：https://github.com/teddysun/across/blob/master/l2tp.sh
[^4]:openwrt的说明：https://openwrt.org/docs/guide-user/services/vpn/ipsec/strongswan/basics
[^5]:PSec/IKEv2 VPN安装脚本 For CentOS/Debian/Ubuntu https://qiaodahai.com/ipsec-ikev2-installation-script-for-centos-debian-ubuntu.html

[6]:https://openwrt.org/inbox/strongswan_certificates