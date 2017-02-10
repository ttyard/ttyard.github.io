---
id: 37
title: XenCenter 无法关机处理办法
date: 2014-07-30T15:21:28+00:00
author: 深海游鱼
layout: post
guid: http://www.wanglijie.cn/?p=37
permalink: '/2014/07/xencenter-%e6%97%a0%e6%b3%95%e5%85%b3%e6%9c%ba%e5%a4%84%e7%90%86%e5%8a%9e%e6%b3%95.html'
views:
  - "317"
categories:
  - 云计算
tags:
  - 云计算-Xen  
---
通过XenCenter无法关机，一直卡在那里，控制台界面也看不到，SSH登陆xen server

[root@xenserver2 ~]# xe vm-list

找到这台挂起的VM对应的UUID

[root@xenserver2 ~]# xe vm-shutdown uuid=627c4220-dd2e-5bf7-4ad1-871187c83933 force=true

发现没用，命令卡在那里

[root@xenserver2 ~]#xe vm-reset-powerstate uuid=627c4220-dd2e-5bf7-4ad1-871187c83933 &#8211;force

依然没用，到网上找一圈，有人说是关机任务被挂起了，取消关机任务再执行关闭即可，于是

[root@xenserver2 ~]#xe task-list 发现是有对应的关机任务，于是输入对应的任务UUID取消之

[root@xenserver2 ~]#xe task-cancel uuid=85f509b3-d240-7dcf-4175-523c839b8145

再执行查看任务列表

<!--more-->[root@xenserver2 ~]#xe task-list 发现任务依然存在，这下无奈了&#8230;&#8230;&#8230;&#8230;!到citrix官方论坛上，发现和我一样问题的老外还不少，找了一圈，终于找到一个靠谱的办法，如下 [root@xenserver2 ~]# xe vm-list

找到这台挂起的VM对应的UUID

[root@xenserver2 ~]# list_domains

找出对应UUID的域ID

[root@xenserver2 ~]# /opt/xensource/debug/destroy_domain -domid XX

这个命令我猜应该是删除这台VM的外联存储（NFS、ISCSI等）<span style="font-size: 16px; line-height: 1.5;">[root@xenserver2 ~]# xe vm-reboot uuid=XXXX &#8211;force </span>

<span style="font-size: 16px; line-height: 1.5;">执行VM重启，搞定！如果控制台还是看不到界面，把XAPI服务重启一下 </span>

<span style="font-size: 16px; line-height: 1.5;">[root@xenserver2 ~]# xe-toolstack-restart </span>

<span style="font-size: 16px; line-height: 1.5;">然后重新连接xen server，VM恢复正常，问题解决！ 原文：http://blog.sina.com.cn/s/blog_7cc2d34401016j9h.html</span>

转载请注明：[自动化运维](http://www.wanglijie.cn) &raquo; [XenCenter 无法关机处理办法](http://www.wanglijie.cn/2014/07/xencenter-%e6%97%a0%e6%b3%95%e5%85%b3%e6%9c%ba%e5%a4%84%e7%90%86%e5%8a%9e%e6%b3%95.html)