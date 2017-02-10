---
id: 215
title: Amazon EC2 RHEL 服务器搭建PPTP VPN与SNAT
date: 2014-11-18T15:21:31+00:00
author: 深海游鱼
layout: post
guid: http://www.aixiuyun.com/?p=215
permalink: '/2014/11/amazon-ec2-ubuntu-%e6%9c%8d%e5%8a%a1%e5%99%a8%e6%90%ad%e5%bb%bapptp-vpn%e4%b8%8esnat.html'
views:
  - "470"
categories:
  - 网络
tags:
  - VPN  
---
首先确认服务器操作系统版本，搭建PPTP的VPN，只需要安装ppp和pptpd连个软件包即可。客户端可直接使用系统自带的PPPoE拨号程序，选择“设置新的连接或网络”->“连接到工作区”->“创建新连接”，依次填写服务器IP、用户名和密码记录拨号成功。
  
话说PPTP的VPN并不安全，

1、验证ppp
  
用cat命令检查是否开启ppp，一般服务器都是开启的，除了特殊的VPS主机之外。

<pre class="prettyprint linenums">[root@localhost1 /]# cat /dev/ppp
cat: /dev/ppp: No such device or address
</pre>

cat出现上面结果，则说明ppp是开启的，可以正常的配置pptp了。

2、安装PPP

<pre class="prettyprint linenums">[root@localhost1 /]# yum -y install ppp
</pre>

iptables一般情况默认都是系统装好后就已经有了，安装iptables是为了做NAT，让PPTP客户端能够通过PPTP服务器上外网。

3、安装PPTP

<pre class="prettyprint linenums">[root@localhost1 ~]# wget http://poptop.sourceforge.net/yum/stable/rhel6/pptp-release-current.noarch.rpm
[root@localhost1 ~]# rpm -ivh pptp-release-current.noarch.rpm
[root@localhost1 ~]# yum update
[root@localhost1 ~]# yum -y pptpd
</pre>

安装PPTP的源，更新yum并安装pptp

4、配置pptp

在最底下添加下面两行，localip是pptp服务器端IP，remoteip是客户端获取的IP地址范围

<pre class="prettyprint linenums">[root@localhost1 /]# vi /etc/pptpd.conf
#如果localip地址段，则每个客户端拨入后，网关的IP会从localip地址池中动态分配
localip 192.168.110.10
remoteip 192.168.110.10-110
</pre>

修改options.pptpd文件，打开后，找到下面字段，并修改成你想要为VPN用户分配的dns服务器

<pre class="prettyprint linenums">#Ubuntu 系统请编辑 vi /etc/ppp/options
[root@localhost1 /]# vi /etc/ppp/options.pptpd
ms-dns 8.8.8.8
ms-dns 8.8.4.4
</pre>

添加vpn的帐号和密码
  
一行添加一个账号，每个帐号需要添加的4个字段，分别为：用户名、服务、密码、分配的ip地址（如果IP为*，则表示随机分配，分配范围采用pptp.conf中的设置）

<pre class="prettyprint linenums">[root@localhost1 /]# vi /etc/ppp/chap-secrets
# client server secret IP addresses
test * test123 *
</pre>

5、开启ipv4转发

<pre class="prettyprint linenums">[root@localhost1 /]# vi /etc/sysctl.conf
net.ipv4.ip_forward = 1
#保存退出，并执行下面命令使内核配置生效：
[root@localhost1 /]# sysctl -p
</pre>

6、配置iptables SNAT
  
配置外网eth0的数据转发，并加入开机启动rc.local中

<pre class="prettyprint linenums">[root@localhost1 /]# iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
[root@localhost1 /]# echo "iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE" &gt;&gt; /etc/rc.local
</pre>

7、启动服务

<pre class="prettyprint linenums">[root@localhost1 /]# service pptpd start
[root@localhost1 /]# service iptables start
</pre>

将服务配置为开机自动启动

<pre class="prettyprint linenums">[root@localhost1 /]# chkconfig pptpd on
[root@localhost1 /]# chkconfig iptables on
</pre>

转载请注明：[自动化运维](http://www.wanglijie.cn) &raquo; [Amazon EC2 RHEL 服务器搭建PPTP VPN与SNAT](http://www.wanglijie.cn/2014/11/amazon-ec2-ubuntu-%e6%9c%8d%e5%8a%a1%e5%99%a8%e6%90%ad%e5%bb%bapptp-vpn%e4%b8%8esnat.html)