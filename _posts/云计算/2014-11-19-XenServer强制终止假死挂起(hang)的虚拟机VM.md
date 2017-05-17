---
id: 223
title: XenServer强制终止假死挂起(hang)的虚拟机VM
date: 2014-11-19T14:52:03+00:00
author: 深海游鱼
layout: post
guid: http://www.wanglijie.cn/?p=223
permalink: '/2014/11/xenserver%e5%bc%ba%e5%88%b6%e7%bb%88%e6%ad%a2%e5%81%87%e6%ad%bb%e6%8c%82%e8%b5%b7hang%e7%9a%84%e8%99%9a%e6%8b%9f%e6%9c%bavm.html'
views:
  - "626"
image: /wp-content/uploads/2014/11/xen.jpg
categories:
  - 云计算
tags:
  - Xen
  - 云计算
---
在XenServer中，经常出现虚拟机挂起假死，使用XenCenter无法关机、重启的情况，这是应为虚拟机（VM）挂起假死所致。造成虚拟机假死的原因很多，如:本身虚拟机系统的原因、XenServer底层XAPI接口的问题。

如果是系统的原因一般强制重启就可以解决；但是xapi有问题，强制重启方法有时是行不通的。

XAPI就是XenServer中的一组管理接口的统称，是XenServer管理的核心，由一系列的toolstack组成。
  
XenCenter通过XAPI来读取XenServer的配置、管理、License的管理、数据库的维护等等，同时也包括如存储（SR）、虚机、虚拟网卡、HA等等所有的功能控制。简而言之，XAPI就是个和底层通信的中间层、接口层。

一般情况下，为了关闭VM或者重启VM，我们推荐这样的操作顺序：

1.进入到VM内，使用系统的关机或者重启功能

通过XenCenter的菜单选择ShutDown或者Restart。虽然这个菜单的实现是通过XenServer tool来控制系统的命令来实现，但是不保证在XenServer Tools工作异常的情况下，导致VM挂起（Hang），而且，这个应该也是VM挂起（XenCenter中VM标志处于黄色状态）的主要原因。尝试通过XenCenter菜单的Force Shutdown和Force Restart来强制操作。

如果这些操作都进行了以后，VM也长时间处于挂起状态，为了让VM能够关机，或者说是强制关机来重置其状态，我们有以下几种解决方法，这些解决方法的危害会逐渐增加。所以，请按顺序尝试：

2.尝试重置VM的电源状态

<pre class="prettyprint linenums">xe vm-reset-powerstate force=true vm=['name-label']
</pre>

3.尝试重启toolstack

<pre class="prettyprint linenums">#重启xapi守护进程
service xapi restart
#或者
xe-toolstack-restart
</pre>

4.尝试destroy domain

<pre class="prettyprint linenums">#首先获取VM的UUID
#[] 内的内容必选
xe vm-list name-label=['your-name-label'] params=uuid
#获取VM的Domain ID
list_domains | grep 'your-uuid'
#尝试重置挂起状态的VM
/opt/xensource/debug/xenops destroy_domain -domid [your-id]
</pre>

5.如果到这里还不行，就只能强制VM进入崩溃状态

<pre class="prettyprint linenums">#首先获取VM的UUID
xe vm-list name-label=['your-name-label'] params=uuid
#获取VM的Domain ID
list_domains | grep
#手动触发VM的Crash机制
/usr/lib/xen/bin/crash_guest
</pre>

注：在Crash VM以后，VM会处于蓝屏状态，这个时候，可以再试试正常的关机或者强制关机命令来关闭虚机。

6.如果连Crash机制都不起作用的情况下，那么就只剩下强制关闭XenServer主机电源一条途径了。

转载请注明：[自动化运维](http://www.wanglijie.cn) &raquo; [XenServer强制终止假死挂起(hang)的虚拟机VM](http://www.wanglijie.cn/2014/11/xenserver%e5%bc%ba%e5%88%b6%e7%bb%88%e6%ad%a2%e5%81%87%e6%ad%bb%e6%8c%82%e8%b5%b7hang%e7%9a%84%e8%99%9a%e6%8b%9f%e6%9c%bavm.html)