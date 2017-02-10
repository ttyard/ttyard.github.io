---
id: 245
title: CentOS6/RedHat6新增网卡识别问题
date: 2014-12-10T11:09:12+00:00
author: 深海游鱼
layout: post
guid: http://www.aixiuyun.com/?p=245
permalink: '/2014/12/centos6redhat6%e6%96%b0%e5%a2%9e%e7%bd%91%e5%8d%a1%e8%af%86%e5%88%ab%e9%97%ae%e9%a2%98.html'
views:
  - "392"
categories:
  - 运维
tags:
  - Linux  
---
使用虚拟机做实验时，有时需要新增网卡设备，有些时候新增的设备并不能正常的识别出来，可以通过下面的方法处理
  
1.添加新硬件，重启udev服务

<pre class="prettyprint linenums" >start_udev
</pre>

2.查看网卡配置信息，确认正确识别

<pre class="prettyprint linenums" >vim  /etc/udev/rules.d/70-persistent-net.rules
# PCI device 0x8086:0x100e (e1000)
SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="08:00:27:de:15:d5", ATTR{type}=="1", KERNEL=="eth*", NAME="eth0"

# PCI device 0x8086:0x100e (e1000)
SUBSYSTEM=="net", ACTION=="add", DRIVERS=="?*", ATTR{address}=="08:00:27:3d:76:c8", ATTR{type}=="1", KERNEL=="eth*", NAME="eth1"
</pre>

3.生成网卡配置文件
  
使用setup或system-config-network-tui添加了网卡配置信息。如果在/etc/sysconfig/network-scripts/找不到新增网卡的配置文件，可以将/etc/sysconfig/networking/devices/下的配置文件直接复制到network-scripts文件夹中。

<pre class="prettyprint linenums" >cp /etc/sysconfig/networking/devices/* /etc/sysconfig/network-scripts/
</pre>

转载请注明：[自动化运维](http://www.wanglijie.cn) &raquo; [CentOS6/RedHat6新增网卡识别问题](http://www.wanglijie.cn/2014/12/centos6redhat6%e6%96%b0%e5%a2%9e%e7%bd%91%e5%8d%a1%e8%af%86%e5%88%ab%e9%97%ae%e9%a2%98.html)