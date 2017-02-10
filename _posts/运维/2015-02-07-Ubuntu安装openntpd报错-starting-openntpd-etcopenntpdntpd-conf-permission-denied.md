---
id: 273
title: 'Ubuntu安装openntpd报错：Starting openntpd: /etc/openntpd/ntpd.conf: Permission denied'
date: 2015-02-07T21:07:17+00:00
author: 深海游鱼
layout: post
guid: http://www.wanglijie.cn/?p=273
permalink: '/2015/02/ubuntu%e5%ae%89%e8%a3%85openntpd%e6%8a%a5%e9%94%99%ef%bc%9astarting-openntpd-etcopenntpdntpd-conf-permission-denied.html'
views:
  - "471"
categories:
  - 运维
tags:
  - Linux
---
在Ubuntu 14.04安装 openntpd时报错：

<pre class="prettyprint linenums" >Setting up openntpd (20080406p-7) ...
Starting openntpd: /etc/openntpd/ntpd.conf: Permission denied
invoke-rc.d: initscript openntpd, action "start" failed.
dpkg: error processing package openntpd (--configure):
 subprocess installed post-installation script returned error exit status 1
Processing triggers for ureadahead (0.100.0-16) ...
Errors were encountered while processing:
 openntpd
E: Sub-process /usr/bin/dpkg returned an error code (1)
</pre>

随后通过下面命令可以解决：

<pre class="prettyprint linenums" >apt-get install openntpd -y
apt-get purge ntp -y
service apparmor restart
service openntpd restart
</pre>

原文链接：https://bugs.launchpad.net/ubuntu/+source/openntpd/+bug/458061

转载请注明：[自动化运维](http://www.wanglijie.cn) &raquo; [Ubuntu安装openntpd报错：Starting openntpd: /etc/openntpd/ntpd.conf: Permission denied](http://www.wanglijie.cn/2015/02/ubuntu%e5%ae%89%e8%a3%85openntpd%e6%8a%a5%e9%94%99%ef%bc%9astarting-openntpd-etcopenntpdntpd-conf-permission-denied.html)