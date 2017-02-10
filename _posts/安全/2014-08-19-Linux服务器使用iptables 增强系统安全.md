---
id: 170
title: Linux服务器使用iptables 增强系统安全
date: 2014-08-19T13:20:38+00:00
author: 深海游鱼
layout: post
guid: http://www.aixiuyun.com/?p=170
permalink: '/2014/08/linux%e6%9c%8d%e5%8a%a1%e5%99%a8%e4%bd%bf%e7%94%a8iptables-%e5%a2%9e%e5%bc%ba%e7%b3%bb%e7%bb%9f%e5%ae%89%e5%85%a8.html'
views:
  - "353"
image: /wp-content/uploads/2014/08/iptables_small-220x150.png
categories:
  - 安全
tags:
  - 系统安全
---
在Linux系统中，通过使用iptables防火墙，能够增强系统的安全性。下面将从CentOS6普通Web服务器入手，介绍iptables相关配置。

1.修改默认iptables规则，禁用所有通讯，修改成如下内容：

<pre class="prettyprint linenums">vim /etc/sysconfig/iptables

:INPUT DROP [0:0]
:FORWARD DROP [0:0]
</pre>

2.规范ping命令

<pre class="prettyprint linenums">-A INPUT -p icmp --icmp-type echo-relay -j ACCEPT
-A INPUT -p icmp --icmp-type destination-unreachable -j ACCEPT
-A INPUT -p icmp --icmp-type time-exceeded -j ACCEPT
#限制ping应用于系统的比特率
-A INPUT -p icmp --icmp-type echo-request -m limit --limit 1/s -j ACCEPT
</pre>

3.阻止可疑私有IP地址
  
如果是双网卡，eth0直连互联网拥有固定IP，则可拒绝私网地址从eth0访问

<pre class="prettyprint linemums">-A INPUT -i eth0 -s 10.0.0.0/8 -j DROP
-A INPUT -i eth0 -s 172.16.0.0/12 -j DROP
-A INPUT -i eth0 -s 192.168.0.0/16 -j DROP
-A INPUT -i eth0 -s 224.0.0.0/4 -j DROP
-A INPUT -i eth0 -s 240.0.0.0/5 -j DROP</pre>

<pre>4.规范SSH的访问
</pre>

<pre class="prettyprint linenums">-A INPUT -i eth0 -p tcp -m tcp --dport 22 -m state --state NEW -j SSH_CHAIN
-A SSH_CHAIN -i eth0 -p tcp -m tcp --dport 22 -m state --state NEW -m recent--update --seconds 60 --hitcount 3 --rttl --name SSH -j DROP
</pre>

转载请注明：[自动化运维](http://www.wanglijie.cn) &raquo; [Linux服务器使用iptables 增强系统安全](http://www.wanglijie.cn/2014/08/linux%e6%9c%8d%e5%8a%a1%e5%99%a8%e4%bd%bf%e7%94%a8iptables-%e5%a2%9e%e5%bc%ba%e7%b3%bb%e7%bb%9f%e5%ae%89%e5%85%a8.html)