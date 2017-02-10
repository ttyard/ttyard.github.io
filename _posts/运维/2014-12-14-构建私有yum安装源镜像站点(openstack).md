---
id: 248
title: 构建私有yum安装源镜像站点(openstack)
date: 2014-12-14T12:04:51+00:00
author: 深海游鱼
layout: post
guid: http://www.aixiuyun.com/?p=248
permalink: '/2014/12/%e6%9e%84%e5%bb%ba%e7%a7%81%e6%9c%89yum%e5%ae%89%e8%a3%85%e6%ba%90%e9%95%9c%e5%83%8f%e7%ab%99%e7%82%b9openstack.html'
views:
  - "952"
categories:
  - 运维
tags:
  - Linux  
---
在本地网络环境中，经常需要安装一个国外的镜像站点的软件包，如rdo openstack，但由于这些站点访问速度很慢，于是萌发了构建第三方yum源的想法，本文将以Centos6构建openstack为例，演示如何自建yum站点。
  
1.安装必要的软件包

<pre class="prettyprint linenums" >#yum源制作工具
yum install yum-utils createrepo yum-plugin-priorities
#Web服务器
yum install httpd
#设置开机启动
chkconfig httpd on
</pre>

2.获取repo文件并使用reposync同步源

<pre class="prettyprint linenums" >#安装openstack yum
yum install -y https://repos.fedorapeople.org/repos/openstack/openstack-icehouse/rdo-release-icehouse-4.noarch.rpm
yum repolist 
repo id             repo name                                      status
base                CentOS-6 - Base                                 6,497+21
epel                Extra Packages for Enterprise Linux 6 - x86_64 11,258+25
extras              CentOS-6 - Extras                                   34+2
foreman             Foreman stable                                       199
foreman-plugins     Foreman stable - plugins                              88
openstack-icehouse  OpenStack Icehouse Repository wanglijie.cn         1,323
puppetlabs-deps     Puppet Labs Dependencies - x86_64                     77
puppetlabs-products Puppet Labs Products - x86_64                        461
updates             CentOS-6 - Updates                                 488+6
repolist: 20,425
#我们要同步openstack-icehouse 这个repo id
#创建openstakc目录
mkdir /var/www/html/openstack/
cd /var/www/html/openstack/
#同步安装源的软件包，由于官方站点速度，需要很久
reposync --repoid=openstack-icehouse 
#同步完成后，创建yum必要的信息	
createrepo –update /var/www/html/openstack/openstack-icehouse
#将站点目录结构与官方保持一直
</pre>

3.制作rdo-release.repo文件

<pre class="prettyprint linenums" >vim /etc/yum.repos.d/rdo-aixiuyun-release.repo
[openstack-icehouse]
name=OpenStack Icehouse Repository wanglijie.cn
baseurl=http://mirrors.wanglijie.cn/openstack/openstack-icehouse
enabled=1
skip_if_unavailable=0
gpgcheck=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-RDO-Icehouse
priority=98
</pre>

4.清除本地源缓存

<pre class="prettyprint linenums" >yum clean all
yum update
</pre>

转载请注明：[自动化运维](http://www.wanglijie.cn) &raquo; [构建私有yum安装源镜像站点(openstack)](http://www.wanglijie.cn/2014/12/%e6%9e%84%e5%bb%ba%e7%a7%81%e6%9c%89yum%e5%ae%89%e8%a3%85%e6%ba%90%e9%95%9c%e5%83%8f%e7%ab%99%e7%82%b9openstack.html)