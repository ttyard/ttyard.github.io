---
id: 279
title: 'PostgreSQL中的log, xlog和clog 说明 [翻译/转]'
date: 2015-03-23T15:26:29+00:00
author: 深海游鱼
layout: post
guid: http://www.wanglijie.cn/?p=279
permalink: '/2015/03/postgresql%e4%b8%ad%e7%9a%84log-xlog%e5%92%8cclog-%e8%af%b4%e6%98%8e-%e7%bf%bb%e8%af%91%e8%bd%ac.html'
views:
  - "577"
categories:
  - 数据库
tags:
  - 数据库-PostgreSQL
---
## **pg_log**

$PGDATA/pg_log是数据库运行活动日志的默认保存目录，它包括错误信息，查询日志以及启动/关闭数据库的信息。当PostgreSQL启动失败时，这里应该是你第一个应该查看的信息。一些Linux发行版以及其他的软件包管理系统会将这个日志目录移到某些地方，比如：/var/log/postgresql

你可以在pg\_log目录里自由地删除、重命名、压缩或者移动文件而不会有什么不好的结果，只要Postgres用户仍然有权限写该目录。如果pg\_log随着许多大文件而膨胀，你可能需要在postgresql.conf里减小你想记录日志的事件。

## pg_xlog

$PGDATA/pg\_xlog是PostgreSQL的事务日志。 这是一些二进制日志文件的集合，文件名类似00000001000000000000008E，它包含最近事务的一些描述数据。这些日志也被用于二进制复制。如果复制、归档或者PITR失败了，当归档正在恢复时，这个目录保存的数据库日志可能会膨胀数GB。这可能会导致你用完你的磁盘空间。不像pg\_log，你不能自由地删除、移动或者压缩这个目录的文件。你甚至不能在没有符号链接到该目录的情况下移动这个目录。删除pg_xlog的文件可能会导致不可恢复的数据库损坏。

如果你发现自己处在这样的情况：你发现有100G大小的文件在pg_xlog目录并且数据也启动不了，并且你已经禁止归档/复制并且尝试清理磁盘空间等任何其他的方式，请做以下两个步骤：

从pg_xlog目录里移动文件到一个备份磁盘或者共享网络驱动器中，也不要删除它们，并且移动一些最老的文件，直到足够允许PostgreSQL启动起来。

## **pg_clog**

$PGDATA/pg\_clog包含了事务的元数据。这种日志用于告诉PostgreSQL哪个事务已经完成、哪个还没有完成。clog是比较小的并且没有任何理由会膨胀，所以，你应该没有任何理由去碰触它。在任何时候你都不应该从pg\_clog里删除文件，如果你这样子做，还不如完全地删除整个数据库目录。缺少clog是不可恢复的。请注意，这意味着，如果你在$PGDATA目录里备份文件，你应该确定同时包含pg\_clog和pg\_xlog，否则你可能会发现你的备份是不可用的。

<a title="原文" href="http://it.toolbox.com/blogs/database-soup/pg_log-pg_xlog-and-pg_clog-45611" target="_blank">原文</a>/<a title="翻译" href="http://dreamer-yzy.github.io/2015/01/21/-%E7%BF%BB%E8%AF%91-PostgreSQL%E4%B8%AD%E7%9A%84log-xlog%E5%92%8Cclog/" target="_blank">翻译</a>

转载请注明：[自动化运维](http://www.wanglijie.cn) &raquo; [PostgreSQL中的log, xlog和clog 说明 [翻译/转]](http://www.wanglijie.cn/2015/03/postgresql%e4%b8%ad%e7%9a%84log-xlog%e5%92%8cclog-%e8%af%b4%e6%98%8e-%e7%bf%bb%e8%af%91%e8%bd%ac.html)