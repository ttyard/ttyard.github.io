---
id: 600
title: Ubuntu 14.04 Linux如何配置静态IP地址和DNS服务器
date: 2016-07-06T13:49:30+00:00
author: 深海游鱼
layout: post
guid: http://www.wanglijie.cn/?p=600
permalink: '/2016/07/ubuntu-14-04-linux%e5%a6%82%e4%bd%95%e9%85%8d%e7%bd%ae%e9%9d%99%e6%80%81ip%e5%9c%b0%e5%9d%80%e5%92%8cdns%e6%9c%8d%e5%8a%a1%e5%99%a8.html'
views:
  - "25"
image: /wp-content/uploads/2016/07/logo-ubuntu_st_no®-black_orange-hex-220x150.png
categories:
  - 运维
tags:
  - Linux
---
本篇属于普及类的文章，将介绍如何在Ubuntu 14.04 Linux操作系统下配置静态的IP地址和DNS服务器。
  
<img class="aligncenter  wp-image-606" src="http://images.wanglijie.cn/public/img/posts/2016/07/logo-ubuntu_st_no®-black_orange-hex.png" alt="logo-ubuntu_st_no®-black_orange-hex" width="425" height="304" />

## 1.配置静态IP地址

在Ubuntu Linux操作系统中，静态IP地址配置文件在/etc/network/interfaces中配置.具体配置格式输入:

**默认采用DHCP方式**,如下文所示:

<pre class="prettyprint linenums">auto eth0
iface eth0 inet dhcp
</pre>

**修改成静态的IP地址**

<pre class="prettyprint linenums">auto eth0
  iface eth0 inet static
  address 192.168.0.150
  netmask 255.255.255.0
  gateway 192.168.0.1
  network 192.168.0.0
  broadcast 192.168.0.255
  dns-nameservers 8.8.8.8 114.114.114.114
</pre>

**重新启动网卡服务或重启电脑均可生效.**

<pre class="prettyprint linenums">#如果您使用SSH远程连接,请不要使用这条命令
$ ip down eth0
$ ip up eth0
</pre>

## 2.配置NAMESERVER

在Ubuntu14.04 中,已经不再运行直接修改/etc/resolv.conf配置文件来添加nameserver,新增的记录一般会在重启后清空.替代方法是你可以/etc/resolvconf/resolv.conf.d/目录下的head 或 base文件达到同样的效果.

<pre class="prettyprint linenums">/etc/resolvconf/resolv.conf.d/head
/etc/network/interfaces
/etc/resolvconf/resolv.conf.d/base
</pre>

注意:

修改/etc/resolvconf/resolv.conf.d/配置文件进行DNS Server需要重启后才能够自动重写resolv.conf配置文件.

转载请注明：[自动化运维](http://www.wanglijie.cn) &raquo; [Ubuntu 14.04 Linux如何配置静态IP地址和DNS服务器](http://www.wanglijie.cn/2016/07/ubuntu-14-04-linux%e5%a6%82%e4%bd%95%e9%85%8d%e7%bd%ae%e9%9d%99%e6%80%81ip%e5%9c%b0%e5%9d%80%e5%92%8cdns%e6%9c%8d%e5%8a%a1%e5%99%a8.html)