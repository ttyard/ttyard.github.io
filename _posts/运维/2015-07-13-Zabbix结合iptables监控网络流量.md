---
id: 344
title: Zabbix结合iptables监控网络流量
date: 2015-07-13T16:27:34+00:00
author: 深海游鱼
layout: post
guid: http://www.wanglijie.cn/?p=344
permalink: '/2015/07/zabbix%e7%bb%93%e5%90%88iptables%e7%9b%91%e6%8e%a7%e7%bd%91%e7%bb%9c%e6%b5%81%e9%87%8f.html'
views:
  - "508"
categories:
  - 运维
tags:
  - 监控

---
在实际生产环境监控体系中，由于个别任务的需要对服务器中对外数据库连接的流量进行情况。为此我使用linux操作系统的iptables结合zabbix定时对网络中特定IP的流量情况进行采集，具体操作如下：

## 1.配置iptables防火墙过滤规则，统计指定IP的INPUT和OUTPUT流量。

<pre class="prettyprint linenums" >#统计INPUT源地址为192.168.1.12 IP的流量
iptables -I INPUT -s 192.168.1.12
#统计OUTPUT目的地址为192.168.1.12 IP的流量
iptables -I OUTPUT -d 192.168.1.12
</pre>

使用iptables命令查看防火墙统计的流量情况，如下:

<pre class="prettyprint linenums" >root@ip-172-31-15-86:/home/bitnami# iptables -nvx -t filter -L
Chain INPUT (policy ACCEPT 9559261 packets, 1806802825 bytes)
    pkts      bytes target     prot opt in     out     source               destination         
 1684184 619432744            all  --  *      *       192.168.1.12        0.0.0.0/0           

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
    pkts      bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 9229817 packets, 2342781143 bytes)
    pkts      bytes target     prot opt in     out     source               destination         
 1927409 467400209            all  --  *      *       0.0.0.0/0            192.168.1.12       
</pre>

其中：
  
-L : 未指定参数的情况下默认输出INPUT、OUTPUT的流量。
  
-x : 用于精确输出流量，单位Byte

## 2.创建Zabbix自定义KEY

<pre class="prettyprint linenums" >vim /etc/zabbix/zabbix_agentd.d/network.flow.conf 
#Network Flow INPUT Total
UserParameter=networkflow.input.bytes.1.12,var=$(sudo /sbin/iptables -nvx -L INPUT -t filter |grep '192.168.1.12' |awk '{print $2}')&& if [ "${var}" != "" ]; then  echo ${var};else echo 0;fi
#Network Flow OUTPUT Total
UserParameter=networkflow.output.bytes.1.12,var=$(sudo /sbin/iptables -nvx -L OUTPUT -t filter |grep '192.168.1.12' |awk '{print $2}')&& if [ "${var}" != "" ]; then  echo ${var};else echo 0;fi

</pre>

## 3.配置zabbix用户执行iptables权限，修改/etc/sudoers

Linux操作系统默认情况下，iptables只能是超级管理员root才能运行。zabbix用户是无权执行任何iptables命令的。在不影响操作系统安全的情况下，可以配置sudoers来让zabbix用户只能执行我们需要的指令。

<pre class="prettyprint linenums" >zabbix ALL=(ALL) NOPASSWD: /sbin/iptables -nvx -L OUTPUT -t filter,/sbin/iptables -nvx -L INPUT -t filter
</pre>

## 4.Zabbix Server配置监控项目

打开需要添加监控项目的主机，新增上面定义的KEY，示例如下所示：
  
[<img src="http://images.wanglijie.cn/public/img/posts/2015/07/ITEN.jpg" alt="ITEN" width="736" height="536" class="aligncenter size-full wp-image-347" srcset="http://images.wanglijie.cn/public/img/posts/2015/07/ITEN.jpg 736w, http://images.wanglijie.cn/public/img/posts/2015/07/ITEN-300x218.jpg 300w" sizes="(max-width: 736px) 100vw, 736px" />](http://images.wanglijie.cn/public/img/posts/2015/07/ITEN.jpg)

完成后保存自定义的KEY，过一会就能够采集到数据了。

转载请注明：[自动化运维](http://www.wanglijie.cn) &raquo; [Zabbix结合iptables监控网络流量](http://www.wanglijie.cn/2015/07/zabbix%e7%bb%93%e5%90%88iptables%e7%9b%91%e6%8e%a7%e7%bd%91%e7%bb%9c%e6%b5%81%e9%87%8f.html)