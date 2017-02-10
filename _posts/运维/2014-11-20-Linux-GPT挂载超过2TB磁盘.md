---
id: 230
title: Linux GPT挂载超过2TB磁盘
date: 2014-11-20T14:51:51+00:00
author: 深海游鱼
layout: post
guid: http://www.wanglijie.cn/?p=230
permalink: '/2014/11/linux-gpt%e6%8c%82%e8%bd%bd%e8%b6%85%e8%bf%872tb%e7%a3%81%e7%9b%98.html'
views:
  - "404"
categories:
  - 运维
tags:
  - Linux  
---
GPT:
  
允许每个磁盘有多达 128 个分区，支持高达 18 千兆兆字节的卷大小，允许将主磁盘分区表和备份磁盘分区表用于冗余，还支持唯一的磁盘和分区 ID (GUID)。

MBR:
  
最大卷为 2 TB (terabytes) ，每个磁盘最多有 4 个主分区（或 3 个主分区，1 个扩展分区和无限制的逻辑驱动器）

<pre class="prettyprint linenums" >#转换成GPT.
parted /dev/sdc mklabel gpt。 
#创建3T的分区。
parted /dev/sdc mkpart primary 0 3000000  
#格式化分区
mkfs -t ext3 /dev/sdc1格式化
#mount就可以用了。
</pre>

转载请注明：[自动化运维](http://www.wanglijie.cn) &raquo; [Linux GPT挂载超过2TB磁盘](http://www.wanglijie.cn/2014/11/linux-gpt%e6%8c%82%e8%bd%bd%e8%b6%85%e8%bf%872tb%e7%a3%81%e7%9b%98.html)