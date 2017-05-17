---
id: 275
title: 'Cloudstack4.4管理节点初始化数据库报错：ERROR 1046 (3D000) at line 285: No database selected'
date: 2015-02-07T21:44:21+00:00
author: 深海游鱼
layout: post
guid: http://www.wanglijie.cn/?p=275
permalink: '/2015/02/cloudstack4-4%e7%ae%a1%e7%90%86%e8%8a%82%e7%82%b9%e5%88%9d%e5%a7%8b%e5%8c%96%e6%95%b0%e6%8d%ae%e5%ba%93%e6%8a%a5%e9%94%99%ef%bc%9aerror-1046-3d000-at-line-285-no-database-selected.html'
views:
  - "786"
categories:
  - 云计算
tags:
  - 云计算
  - CloudStack
---
安装管理节点，初始化数据库配置时报错，经过查询是已修复的BUG，可通过修改初始化数据库脚本，在表名前加上数据库名\`cloud\`即可修复。
  
vim /usr/share/cloudstack-management/setup/create-schema-premium.sql
  
分别修改病添加 \`cloud\`.
  
line 299
   
\`cloud\`.\`netapp_pool\` (\`id\`) ON DELETE CASCADE
  
line318
  
\`cloud\`.\`netapp_volume\` (\`id\`)

原文链接：https://github.com/apache/cloudstack/commit/c368d3b6eeb41efe508bb0d3e0abe3a4ca5bb8e2

root@csm:~/CloudStack# cloudstack-setup-databases cloud:cloud@localhost &#8211;deploy-as=root:root
  
Mysql user name:cloud [ OK ]
  
Mysql user password:\***\*** [ OK ]
  
Mysql server ip:localhost [ OK ]
  
Mysql server port:3306 [ OK ]
  
Mysql root user name:root [ OK ]
  
Mysql root user password:\***\*** [ OK ]
  
Checking Cloud database files &#8230; [ OK ]
  
Checking local machine hostname &#8230; [ OK ]
  
Checking SELinux setup &#8230; [ OK ]
  
Detected local IP address as 192.168.122.49, will use as cluster management server node IP[ OK ]
  
Preparing /etc/cloudstack/management/db.properties [ OK ]
  
Applying /usr/share/cloudstack-management/setup/create-database.sql [ OK ]
  
Applying /usr/share/cloudstack-management/setup/create-schema.sql [ OK ]
  
Applying /usr/share/cloudstack-management/setup/create-database-premium.sql [ OK ]
  
Applying /usr/share/cloudstack-management/setup/create-schema-premium.sql 

We apologize for below error:
  
\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***
  
Encountering an error when executing mysql script
  
&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;-
  
table:
  
/usr/share/cloudstack-management/setup/create-schema-premium.sql

Error:
  
ERROR 1046 (3D000) at line 285: No database selected

Sql parameters:
  
{&#8216;passwd&#8217;: &#8216;root&#8217;, &#8216;host&#8217;: &#8216;localhost&#8217;, &#8216;user&#8217;: &#8216;root&#8217;, &#8216;port&#8217;: 3306}
  
&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;-

\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***\***
  
Please run:

cloud-setup-database -h

for full help

转载请注明：[自动化运维](http://www.wanglijie.cn) &raquo; [Cloudstack4.4管理节点初始化数据库报错：ERROR 1046 (3D000) at line 285: No database selected](http://www.wanglijie.cn/2015/02/cloudstack4-4%e7%ae%a1%e7%90%86%e8%8a%82%e7%82%b9%e5%88%9d%e5%a7%8b%e5%8c%96%e6%95%b0%e6%8d%ae%e5%ba%93%e6%8a%a5%e9%94%99%ef%bc%9aerror-1046-3d000-at-line-285-no-database-selected.html)