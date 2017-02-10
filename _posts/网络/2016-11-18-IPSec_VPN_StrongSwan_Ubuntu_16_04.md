---
id: 283
title: IPSec VPN虚拟专网技术之StrongSwan
date: 2016-11-18T13:06:28+00:00
author: 深海游鱼
layout: post
categories:
  - 网络
tags: 
  - VPN
---

本文将主要介绍基于Linux操作系统平台搭建IPSec VPN虚拟专网

## IPSec VPN虚拟专网技术之StrongSwan

### 1.IPSec概述

### 2.基础环境准备

安装编译环境

```
apt-get install build-essential
```

编译所需要的软件

```
apt-get install libgmp10 libgmp3-dev libssl-dev pkg-config libpcsclite-dev libpam0g-dev
```

### 3.安装配置StrongSwan

#### 3.1 编译安装StrongSwan
```
wget https://download.strongswan.org/strongswan-5.5.0.tar.gz
tar -xzvf strongswan-5.5.0.tar.gz && cd strongswan-5.5.0
./configure --prefix=/usr --sysconfdir=/etc  --enable-openssl --enable-nat-transport --disable-mysql --disable-ldap  --disable-static --enable-shared --enable-md4 --enable-eap-mschapv2 --enable-eap-aka --enable-eap-aka-3gpp2  --enable-eap-gtc --enable-eap-identity --enable-eap-md5 --enable-eap-peap --enable-eap-radius --enable-eap-sim --enable-eap-sim-file --enable-eap-simaka-pseudonym --enable-eap-simaka-reauth --enable-eap-simaka-sql --enable-eap-tls --enable-eap-tnc --enable-eap-ttls
make && make install
```

#### 3.2 配置StrongWan证书

#### 3.2.1.生成x.509 CA根证书

```
mkdir /etc/ipsec.d/p12
cd /etc/ipsec.d
ipsec pki --gen --outform pem > private/caKey.pem
ipsec pki --self --in private/caKey.pem --dn "C=CN, O=Pactera, CN=Pactera VPN Root CA" --ca --outform pem > cacerts/caCert.pem
```

#### 3.2.2.生成服务器证书

```
ipsec pki --gen --outform pem > private/serverKey.pem
ipsec pki --pub --in private/serverKey.pem | ipsec pki --issue --cacert cacerts/caCert.pem --cakey private/caKey.pem --dn "C=CN, O=IPSec VPN, CN=vpn.pactera.top" --san="vpn.pactera.top" --san="vpn.wanglijie.cn" --flag serverAuth --flag ikeIntermediate --outform pem > certs/serverCert.pem
```

#### 3.2.3.创建登录客户端证书

```
ipsec pki --gen --outform pem > private/client1Key.pem
ipsec pki --pub --in private/client1Key.pem | ipsec pki --issue --cacert cacerts/caCert.pem --cakey private/caKey.pem --dn "C=CN, O=IPSec VPN, CN=client1" --outform pem > certs/client1Cert.pem

```

生成的PEM证书无法直接导入的Windows和Android设备中，需要将客户端证书公钥/私钥/CA证书合并到p12文件中

```
openssl pkcs12 -export -inkey private/client1Key.pem -in certs/client1Cert.pem -name "client1" -certfile cacerts/caCert.pem -caname "Pactera VPN Root CA" -out p12/client1Cert.p12
```

#### 3.3.配置StrongSwan
```
mv /etc/ipsec.conf /etc/ipsec.conf.bak
vim /etc/ipsec.conf
config setup
    strictcrlpolicy=no
    uniqueids=no #允许多设备同时在线
conn windowsphone
    keyexchange=ikev2
    ike=aes256-sha1-modp1024!
    esp=aes256-sha1!
    dpdaction=clear
    dpddelay=300s
    rekey=no
    left=%defaultroute
    leftsubnet=0.0.0.0/0
    leftauth=pubkey
    leftcert=serverCert.pem
    leftid="C=CN, O=IPSec VPN, CN=vpn.pactera.top"
    right=%any
    rightsourceip=10.11.1.0/24 #为客户端分配的虚拟地址池
    rightauth=eap-mschapv2
    rightsendcert=never
    eap_identity=%any
    auto=add
conn osx10-ios9-ikev2
    keyexchange=ikev2
    leftid="C=CN, O=IPSec VPN, CN=vpn.pactera.top"
    rightid="*@xxx.org"
    leftsendcert=always
```

#### 3.4.IPSec 密码文件

```
vim /etc/ipsec.secrets
: RSA serverKey.pem
client1 : EAP "client1Password"
wp设备名称\用户名2 : EAP "密码2"  #仅对windowsphone8.1设备
```

对于windowsphone8.1，在客户端输入的用户名发送到服务器显示为设备名称\用户名的形式，故认证需加上设备名称,设备名称在设置-关于-手机信息 中查看

#### 3.5.添加DNS服务器IP

```
vim /etc/strongswan.conf
```

加入分配的dns

```
charon {
    dns1 = 8.8.8.8
    dns2 = 208.67.222.222
}
```

### 4.配置 Iptables 转发
```
iptables -A INPUT -p udp --dport 500 -j ACCEPT
iptables -A INPUT -p udp --dport 4500 -j ACCEPT
echo 1 > /proc/sys/net/ipv4/ip_forward
iptables -t nat -A POSTROUTING -s 10.11.1.0/24 -o eth0 -j MASQUERADE  
```

地址与上面地址池对应

```
iptables -A FORWARD -s 10.11.1.0/24 -j ACCEPT     #同上
```
为避免VPS重启后NAT功能失效，可以把如上5行命令添加到 /etc/rc.local 文件中，添加在exit那一行之前即可。

### 5.启动strongswan:

后台运行: ipsec start

滚动日志: ipsec start --nofork


https://gist.github.com/losisli/11081793

https://zh.opensuse.org/index.php?title=SDB:Setup_Ipsec_VPN_with_Strongswan&variant=zh

https://wiki.archlinux.org/index.php/StrongSwan_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)

https://wiki.archlinux.org/index.php/Openswan_L2TP/IPsec_VPN_client_setup

https://raymii.org/s/tutorials/IPSEC_vpn_with_Ubuntu_15.10.html