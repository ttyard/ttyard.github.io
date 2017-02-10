---
id: 32
title: 云计算网络虚拟化之Open vSwitch
date: 2014-07-28T22:24:30+00:00
author: 深海游鱼
layout: post
guid: http://www.aixiuyun.com/?p=32
permalink: '/2014/07/%e4%ba%91%e8%ae%a1%e7%ae%97%e7%bd%91%e7%bb%9c%e8%99%9a%e6%8b%9f%e5%8c%96%e4%b9%8bopen-vswitch.html'
views:
  - "508"
categories:
  - 云计算
tags:  
  - "云计算-网络虚拟化"
---
计算，存储，网络，安全，是构建任何大型数据中心都绕不过去的四个问题。云也不例外。在这个风起云涌的云时代，各厂商赛马般发布层出不穷的新技术，着实让我们目不暇接。很多人昨天刚玩过Xen，今天看到Redhat宣称KVM是其新的战略方向，又忍不住把KVM拿来折腾一番。大家习惯性地把注意力都放在了“计算”上，积累了不少“服务器虚拟化”的经验，却不知不觉冷落了其余三个方面。国外同行们热议Software Defined Network（SDN）和OpenFlow这些因为云而瞬间火爆的技术时。

<!--more-->


  
**什么是Open vSwitch？它能给云带来什么？**
  
官网首页精炼地回答了这个问题：（翻译自http://openvswitch.org/）
  
Open vSwitch的目标，是做一个具有产品级质量的多层虚拟交换机。通过可编程扩展，可以实现大规模网络的自动化（配置、管理、维护）。它支持现有标准管理接口和协议（比如netFlow，sFlow，SPAN，RSPAN，CLI，LACP，802.1ag等，熟悉物理网络维护的管理员可以毫不费力地通过Open vSwitch转向虚拟网络管理）。

<img class="alignnone  wp-image-33" src="http://www.wanglijie.cn/wp-content/uploads/2014/07/213512822-300x207.jpg" alt="213512822" width="426" height="294" srcset="http://www.wanglijie.cn/wp-content/uploads/2014/07/213512822-300x207.jpg 300w, http://www.wanglijie.cn/wp-content/uploads/2014/07/213512822.jpg 465w" sizes="(max-width: 426px) 100vw, 426px" />

<p style="text-align: center;">
  图一：Open vSwitch示意图
</p>

官网的描述精准而抽象，来点通俗的，Open vSwitch是一个由Nicira Networks主导的开源项目，通过运行在虚拟化平台上的虚拟交换机，为本台物理机上的VM提供二层网络接入， 跟云中的其它物理交换机一样工作在Layer 2层。Open vSwitch充分考虑了在不同虚拟化平台间的移植性，采用平台无关的C语言开发。最为人民群众喜闻乐见的是，它遵循Apache2.0许可，不论你是自用还是商用都OK。而它的同类产品VMware的vDS（vSphere Distributed VirtualSwitch），Cisco的Nexus 1000V都是收费的。更重要的是，虽然免费，其产品质量却深得信赖。在2010年Open vSwitch 1.0.0发布之前，Citrix就宣布在XenServer中将其作为默认组件。关于这一点，也许Nicira的身世可以给出一些解释，它的投资人里有Diane Greene（VMware联合创始人）和Andy Rachleff（Benchmark Capital联合创始人），经常听朋友谈起的几个VMware网络大牛也加盟其中，是目前硅谷最炙手可热的SDN创业公司之一。

既然Citrix的企业级虚拟化产品都装备了Open vSwitch，我们还有什么理由怀疑它的可靠性呢？更何况它还是开源的！

你可能会问，我为什么有必要在自己的云架构中使用它呢？它能给我的云带来什么？

OK。需求决定一切，如果你只是自己搞一台Host，在上面虚拟几台VM做实验。或者小型创业公司，通过在五台十台机器上的虚拟化，创建一些VM给公司内部开发测试团队使用。那么对你而言，网络虚拟化的迫切性并不强烈。也许你更多考虑的，是VM的可靠接入：和物理机一样有效获取网络连接，能够RDP访问。Linux Kernel自带的桥接模块就可以很好的解决这一问题。原理上讲，正确配置桥接，并把VM的virtual nic连接在桥接器上就OK啦。很多虚拟化平台的早期解决方案也是如此，自动配置并以向用户透明的方式提供虚拟机接入。如果你是OpenStack的fans，那Nova就更好地帮你完成了一系列网络接入设置。

但是，如果你是大型数据中心的网络管理员，一朵没有网络虚拟化支持的云，将是无尽的噩梦。

在传统数据中心中，网络管理员习惯了每台物理机的网络接入均可见并且可配置。通过在交换机某端口的策略配置，可以很好控制指定物理机的网络接入，访问策略，网络隔离，流量监控，数据包分析，Qos配置，流量优化等。

有了云，网络管理员仍然期望能以per OS/per port的方式管理。如果没有网络虚拟化技术的支持，管理员只能看到被桥接的物理网卡，其上川流不息地跑着n台VM的数据包。仅凭物理交换机支持，管理员无法区分这些包属于哪个OS哪个用户，只能望云兴叹乎？简单列举常见的几种需求，Open vSwitch现有版本很好地解决了这些需求。

需求一：网络隔离。物理网络管理员早已习惯了把不同的用户组放在不同的VLAN中，例如研发部门、销售部门、财务部门，做到二层网络隔离。Open vSwitch通过在host上虚拟出一个软件交换机，等于在物理交换机上级联了一台新的交换机，所有VM通过级联交换机接入，让管理员能够像配置物理交换机一样把同一台host上的众多VM分配到不同VLAN中去；

需求二：QoS配置。在共享同一个物理网卡的众多VM中，我们期望给每台VM配置不同的速度和带宽，以保证核心业务VM的网络性能。通过在Open vSwitch端口上，给各个VM配置QoS，可以实现物理交换机的traffic queuing和traffic shaping功能。

需求三：流量监控，Netflow，sFlow。物理交换机通过xxFlow技术对数据包采样，记录关键域，发往Analyzer处理。进而实现包括网络监控、应用软件监控、用户监控、网络规划、安全分析、会计和结算、以及网络流量数据库分析和挖掘在内的各项操作。例如，NetFlow流量统计可以采集的数据非常丰富，包括：数据流时戳、源IP地址和目的IP地址、 源端口号和目的端口号、输入接口号和输出接口号、下一跳IP地址、信息流中的总字节数、信息流中的数据包数量、信息流中的第一个和最后一个数据包时戳、源AS和目的AS，及前置掩码序号等。

xxFlow因其方便、快捷、动态、高效的特点，为越来越多的网管人员所接受，成为互联网安全管理的重要手段，特别是在较大网络的管理中，更能体现出其独特优势。

没错，有了Open vSwitch，作为网管的你，可以把xxFlow的强大淋漓尽致地应用在VM上！

需求四：数据包分析，Packet Mirror。物理交换机的一大卖点，当对某一端口的数据包感兴趣时（for trouble shooting , etc），可以配置各种span（SPAN, RSPAN, ERSPAN），把该端口的数据包复制转发到指定端口，通过抓包工具进行分析。Open vSwitch官网列出了对SPAN, RSPAN, and GRE-tunneled mirrors的支持。

关于具体功能，就不一一赘述了，感兴趣的童鞋可以参考官网功能列表：http://openvswitch.org/features/

只是在Open vSwitch上实现物理交换机的现有功能？那绝对不是Nicira的风格。

云中的网络，绝不仅仅需要传统物理交换机已有的功能。云对网络的需求，推动了Software Defined Network越来越火。而在各种SDN解决方案中，OpenFlow无疑是最引人瞩目的。Flow Table + Controller的架构，为新服务新协议提供了绝佳的开放性平台。Nicira把对Openflow的支持引入了Open vSwitch。引入以下模块：
  
ovs-openflowd &#8212; OpenFlow交换机；
  
ovs-controller &#8212; OpenFlow控制器；
  
ovs-ofctl &#8212; Open Flow 的命令行配置接口；
  
ovs-pki &#8212; 创建和管理公钥框架；
  
tcpdump的补丁 &#8212; 解析OpenFlow的消息；

[<img class="alignnone size-medium wp-image-34" src="http://www.wanglijie.cn/wp-content/uploads/2014/07/213438993-300x294.jpg" alt="213438993" width="300" height="294" srcset="http://www.wanglijie.cn/wp-content/uploads/2014/07/213438993-300x294.jpg 300w, http://www.wanglijie.cn/wp-content/uploads/2014/07/213438993.jpg 519w" sizes="(max-width: 300px) 100vw, 300px" />](http://www.wanglijie.cn/wp-content/uploads/2014/07/213438993.jpg)

需求决定产品，正是由于在企业级云中，需要各种丰富的网络功能，VMware才于n年前就推出了vSwitch、vDS等虚拟交换机。正是看到了云中的网络是一块大市场，Cisco才与VMware紧密合作，以partner的形式基于VMware kernel API开发出了自己的分布式虚拟交换机Nexus 1000V（功能对应于VMware的vDS）。可惜的是，这两款产品都是收费的。Citrix倒是基于Open vSwitch快速追赶，推出了自己的Distributed Virtual Switch解决方案。但是不好意思，也是收费的。开源云的标杆OpenStack去年下半年推出了一项宏大的计划，启动了Quantum项目，志在通过引入Open vSwitch，为Open Stack Network模块勾勒出“Connectivity as a service”的动人前景。有时间的话，会再单独开一篇文章讨论。

原文链接：http://bengo.blog.51cto.com/4504843/791213

转载请注明：[自动化运维](http://www.wanglijie.cn) &raquo; [云计算网络虚拟化之Open vSwitch](http://www.wanglijie.cn/2014/07/%e4%ba%91%e8%ae%a1%e7%ae%97%e7%bd%91%e7%bb%9c%e8%99%9a%e6%8b%9f%e5%8c%96%e4%b9%8bopen-vswitch.html)