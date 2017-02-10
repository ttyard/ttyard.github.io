---
id: 27
title: REHL/CentOS安装phpMyadmin
date: 2014-06-11T15:34:38+00:00
author: 深海游鱼
layout: post
guid: http://www.wanglijie.cn/?p=27
permalink: '/2014/06/rehlcentos%e5%ae%89%e8%a3%85phpmyadmin.html'
views:
  - "334"
categories:
  - 运维
tags:
  - Linux
---
我们使用CentOS系统RPMForge软件库的phpMyAdmin,而不是官方的库：所以需要导入RPMForge的GPG密钥：
  
rpm &#8211;import http://dag.wieers.com/rpm/packages/RPM-GPG-KEY.dag.txt
  
x86_64系统：
  
yum install http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.2-2.el6.rf.x86_64.rpm

在i386系统：
  
yum install http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.2-2.el6.rf.i686.rpm<!--more-->

安装phpmyadmin
  
yum install phpmyadmin

现在我们可以设置phpMyAdmin，了我们可以改变Apache的配置来让phpMyAdmin不仅仅只能从localhost登录。

vi /etc/httpd/conf.d/phpmyadmin.conf

<Directory &#8220;/usr/share/phpmyadmin&#8221;>
  
Order Deny,Allow
  
Deny from all
  
\# Allow from 127.0.0.1
  
Allow from all
  
</Directory>

&nbsp;

修改phpMyAdmin的身份验证方式为http：
  
vi /usr/share/phpmyadmin/config.inc.php

/\* Authentication type \*/
  
$cfg\[&#8216;Servers&#8217;\]\[$i\][&#8216;auth_type&#8217;] = &#8216;http&#8217;;

转载请注明：[自动化运维](http://www.wanglijie.cn) &raquo; [REHL/CentOS安装phpMyadmin](http://www.wanglijie.cn/2014/06/rehlcentos%e5%ae%89%e8%a3%85phpmyadmin.html)