---
id: 128
title: '解决CentOS“Zabbix discoverer processes 75% busy”的问题'
date: 2014-08-11T23:22:40+00:00
author: 深海游鱼
layout: post
guid: http://www.wanglijie.cn/?p=128
permalink: '/2014/08/%e8%a7%a3%e5%86%b3zabbix%e6%80%a7%e8%83%bd%e7%9b%91%e6%8e%a7%e7%b3%bb%e7%bb%9fdiscoverer-processes-more-than-75-busy%e7%9a%84%e9%97%ae%e9%a2%98.html'
views:
  - "846"
categories:
  - 运维
tags:
  - "监控"  
---
[<img class="size-medium wp-image-146 aligncenter" src="http://images.wanglijie.cn/public/img/posts/2014/08/Zabbix-2-2-300x124.png" alt="zabbix" width="300" height="124" srcset="http://images.wanglijie.cn/public/img/posts/2014/08/Zabbix-2-2-300x124.png 300w, http://images.wanglijie.cn/public/img/posts/2014/08/Zabbix-2-2.png 728w" sizes="(max-width: 300px) 100vw, 300px" />](http://images.wanglijie.cn/public/img/posts/2014/08/Zabbix-2-2.png)
  
在使用Zabbix过程中，当开启自动发现协议后，频繁出现“ Zabbix discoverer processes more than 75% busy”的报警信息，如下：

<pre class="prettyprint">Trigger: Zabbix discoverer processes more than 75% busy
Trigger status: PROBLEM
Trigger severity: Average
Trigger IP: 61.172.253.59
Item values:
1. Zabbix busy discoverer processes, in % (192.168.1.222:zabbix[process,discoverer,avg,busy]): 100 %
Original event ID: 4690
</pre>

网上找了一下，导致报警的主要原因有很多：
  
1.支撑Zabbix的MySQL卡住了
  
2.Zabbix服务器的IO卡住了都有可能
  
3.Zabbix进程分配到内存不足
  
4.目标服务器网络不通
  
于是，考虑通过增加Zabbix Server启动时初始化进程的数量，直接增加轮询的负载量，避免这种报错。
  
修改/etc/zabbix/zabbix_server.conf，找到StartPollers

<pre class="prettyprint linenums">### Option: StartPollers
#       Number of pre-forked instances of pollers.
#
# Mandatory: no
# Range: 0-1000
# Default:
 StartPollers=5</pre>

根据系统硬件配置，可以设置成更高的数值。还有一种解决办法，就是定时重启一下Zabbix Server服务。可以通过定时脚本来配置，如下所示：

<pre class="prettyprint linenums">crontab -e
@daily service zabbix-server restart &gt; /dev/null 2&gt;&1
</pre>

转载请注明：[自动化运维](http://www.wanglijie.cn) &raquo; [解决CentOS“Zabbix discoverer processes 75% busy”的问题](http://www.wanglijie.cn/2014/08/%e8%a7%a3%e5%86%b3zabbix%e6%80%a7%e8%83%bd%e7%9b%91%e6%8e%a7%e7%b3%bb%e7%bb%9fdiscoverer-processes-more-than-75-busy%e7%9a%84%e9%97%ae%e9%a2%98.html)