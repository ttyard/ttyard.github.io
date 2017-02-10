---
id: 239
title: Ubuntu 14.04 Server优化与使用问题解决
date: 2014-12-04T13:33:03+00:00
author: 深海游鱼
layout: post
guid: http://www.aixiuyun.com/?p=239
permalink: /2014/12/ubuntu-14-04-network-interface-name-p5p1-to-ethx.html
views:
  - "495"
categories:
  - 运维
tags:
  - Linux
---
一、修改默认网卡名称 p5p1 to ethx

在安装完最新版的Ubuntu 14.04 Server版后，发现网卡接口名称不是ethx而是p5p1，这样很是不方便，于是将其名称改回了传统的命名方式。具体如下：
  
**1.修改/etc/default/grub，添加下面的内容：**

<pre class="prettyprint linenums">GRUB_CMDLINE_LINUX_DEFAULT="net.ifnames=0 biosdevname=0"
</pre>

**2.更新grub.cfg文件，并重新启动服务器**

<pre class="prettyprint linenums">grub-mkconfig -o /boot/grub/gru
</pre>

二、网络接口service networking重启网卡无效
  
Ubuntu 14.04传统的service重启和停止网络已经不再支持了，使用service networking start/stop/restart命令无效，导致使用networking 无法正常重启网卡服务，可以尝试使用下面的命令：

<pre class="prettyprint linenums" >#启动网卡
ifup eth0
#关闭网卡
ifdown eth0
</pre>

或者使用Ubuntu 13.10的networking进行替换
  
https://github.com/metral/restore_networking

转载请注明：[自动化运维](http://www.wanglijie.cn) &raquo; [Ubuntu 14.04 Server优化与使用问题解决](http://www.wanglijie.cn/2014/12/ubuntu-14-04-network-interface-name-p5p1-to-ethx.html)