---
id: 233
title: Redhat/CentOS6系统使用ntfs-3g挂载NTFS分区
date: 2014-12-02T10:50:52+00:00
author: 深海游鱼
layout: post
guid: http://www.wanglijie.cn/?p=233
permalink: '/2014/12/redhatcentos6%e7%b3%bb%e7%bb%9f%e6%8c%82%e8%bd%bdntfs%e5%88%86%e5%8c%ba.html'
views:
  - "345"
categories:
  - 运维
tags:
  - Linux  
---
正常情况下，REDHAT/CENTOS系统安装完成后，是无法打开原磁盘的NTFS格式的分区，可以通过安装3g-ntfs工具，实现打开ntfs分区。
  
1.首先下载3g-ntfs软件包

<pre class="prettyprint linenums" >http://rpmfind.net/linux/rpm2html/search.php?query=ntfs-3g
</pre>

2.安装下载的软件包

<pre class="prettyprint linenums" >rpm -ivh ntfs-3g-2014.2.15-8.el6.x86_64.rpm
</pre>

3.挂载分区到指定目录
  
可以通过mount命令，手动挂载分区

<pre class="prettyprint linenums" >mkdir /media/data
mount -t ntfs-3g  /dev/sda2 /media/data/
</pre>

4.配置fstab自动挂载分区

<pre class="prettyprint linenums" >vim /etc/fstab
#在文件最后加入下面的命令
/media/data        /dev/sda2               ntfs-3g defaults        1       1
</pre>

转载请注明：[自动化运维](http://www.wanglijie.cn) &raquo; [Redhat/CentOS6系统使用ntfs-3g挂载NTFS分区](http://www.wanglijie.cn/2014/12/redhatcentos6%e7%b3%bb%e7%bb%9f%e6%8c%82%e8%bd%bdntfs%e5%88%86%e5%8c%ba.html)