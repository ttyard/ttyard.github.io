---
id: 314
title: Kali Linux USB多分区持久存储设备
date: 2015-06-27T23:37:05+00:00
author: 深海游鱼
layout: post
guid: http://www.aixiuyun.com/?p=314
permalink: '/2015/06/kali-linux-usb%e5%a4%9a%e5%88%86%e5%8c%ba%e6%8c%81%e4%b9%85%e5%ad%98%e5%82%a8%e8%ae%be%e5%a4%87.html'
views:
  - "675"
image: /wp-content/uploads/2015/06/kali-multi-usb-persistence-220x150.png
categories:
  - 安全
tags:
  - 系统安全
---
Kali Linux是一款 集合多种安全测试工具的Linux发行版本，本文将介绍如何在USB移动存储设备中建立“加密分区”和“非加密普通分区”用来持久保存系统运行时创建的数据。如下图所示：
  
[<img class="aligncenter size-full wp-image-315" src="http://www.wanglijie.cn/wp-content/uploads/2015/06/kali-multi-usb-persistence.png" alt="kali-multi-usb-persistence" width="2991" height="1475" srcset="http://www.wanglijie.cn/wp-content/uploads/2015/06/kali-multi-usb-persistence.png 2991w, http://www.wanglijie.cn/wp-content/uploads/2015/06/kali-multi-usb-persistence-300x148.png 300w, http://www.wanglijie.cn/wp-content/uploads/2015/06/kali-multi-usb-persistence-1024x505.png 1024w" sizes="(max-width: 2991px) 100vw, 2991px" />](http://www.wanglijie.cn/wp-content/uploads/2015/06/kali-multi-usb-persistence.png)
  
为了完成本次工作，需要准备kali linux iso镜像、U盘8G或16GB。本文使用32GB的U盘。

## 1.创建Kali Linux U盘启动分区

首先需要从www.kali.org下载kali Linux ISO镜像，将干净的U盘插入Linux系统中。使用DD命令将ISO镜像写入U盘，创建启动分区，如：

<pre class="prettyprint linenums">dd if=kali-linux-1.0.9a-amd64.iso of=/dev/sdb bs=1M
</pre>

完成，U盘将会有一个可用于启动Kali Linux的启动分区，用于live 方式启动，在此启动方式的系统数据将不会被保存。

## 2.创建加密工作分区

在U盘中使用parted创建7GB的普通非加密的分区，并标记为work.

<pre class="prettyprint linenums">root@kali:~# parted
GNU Parted 2.3
Using /dev/sda
Welcome to GNU Parted! Type 'help' to view a list of commands.

(parted) print devices
/dev/sda (480GB)
/dev/sdb (31.6GB)

(parted) select /dev/sdb
Using /dev/sdb

(parted) print
Model: SanDisk SanDisk Ultra (scsi)
Disk /dev/sdb: 31.6GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos

Number Start End Size Type File system Flags
1 32.8kB 2988MB 2988MB primary boot, hidden
2 2988MB 3053MB 64.9MB primary fat16

(parted) mkpart primary 3053 10000
(parted) quit
Information: You may need to update /etc/fstab.
</pre>

磁盘分区创建完成后，使用LUKS进行加密、格式化，并新增标记：

<pre class="prettyprint linenums">cryptsetup --verbose --verify-passphrase luksFormat /dev/sdb3
cryptsetup luksOpen /dev/sdb3 my_usb
mkfs.ext3 /dev/mapper/my_usb
e2label /dev/mapper/my_usb persistence
</pre>

在分区中创建persistence.conf，让在重启数据持久保存。

<pre class="prettyprint linenums">mkdir -p /mnt/my_usb
mount /dev/mapper/my_usb /mnt/my_usb
echo "/ union" &gt; /mnt/my_usb/persistence.conf
umount /dev/mapper/my_usb
cryptsetup luksClose /dev/mapper/my_usb
</pre>

执行完上面的步骤，如果正确的情况下，将创建了一个常规分区、并对分区的中数据进行了加密，在选择该分区启动时，将要求提供分区的密码。

## 3.创建普通5GB分区

接下来，我们将创建一个普通的非加密分区。

<pre class="prettyprint linenums">root@kali:~# parted /dev/sdb
GNU Parted 2.3
Using /dev/sdb
Welcome to GNU Parted! Type 'help' to view a list of commands.
(parted) print
Model: SanDisk SanDisk Ultra (scsi)
Disk /dev/sdb: 31.6GB
Sector size (logical/physical): 512B/512B
Partition Table: msdos

Number Start End Size Type File system Flags
1 32.8kB 2988MB 2988MB primary boot, hidden
2 2988MB 3053MB 64.9MB primary fat16
3 3053MB 10.0GB 6947MB primary

(parted) mkpart primary 10000 15000
(parted) quit
Information: You may need to update /etc/fstab.
</pre>

为分区新增标签，并命名为：work

<pre class="prettyprint linenums">mkfs.ext3 /dev/sdb4
e2label /dev/sdb4 work

mkdir -p /mnt/usb
mount /dev/sdb4 /mnt/usb
echo "/ union" &gt; /mnt/usb/persistence.conf
umount /mnt/usb
</pre>

4.U盘启动Kali Linux
  
插入U盘后，需要选择从U盘启动，在出现启动界面boot menu，按TAB键编辑启动菜单。通过输入不同的persistence-label=work 或 persistence-label=persistence来启动普通分区或加密分区。
  
[<img class="aligncenter size-full wp-image-316" src="http://www.wanglijie.cn/wp-content/uploads/2015/06/persistence-work.png" alt="persistence-work" width="1278" height="610" srcset="http://www.wanglijie.cn/wp-content/uploads/2015/06/persistence-work.png 1278w, http://www.wanglijie.cn/wp-content/uploads/2015/06/persistence-work-300x143.png 300w, http://www.wanglijie.cn/wp-content/uploads/2015/06/persistence-work-1024x489.png 1024w" sizes="(max-width: 1278px) 100vw, 1278px" />
  
](http://www.wanglijie.cn/wp-content/uploads/2015/06/persistence-work.png) 
  
<a href="https://www.offensive-security.com/kali-linux/usb-multiple-persistent-stores/" target="_blank">原文链接</a>

转载请注明：[自动化运维](http://www.wanglijie.cn) &raquo; [Kali Linux USB多分区持久存储设备](http://www.wanglijie.cn/2015/06/kali-linux-usb%e5%a4%9a%e5%88%86%e5%8c%ba%e6%8c%81%e4%b9%85%e5%ad%98%e5%82%a8%e8%ae%be%e5%a4%87.html)