---
id: 567
title: Elasticsearch之Shield认证与权限管理
date: 2016-06-30T16:50:44+00:00
author: 深海游鱼
layout: post
guid: http://www.wanglijie.cn/?p=567
permalink: '/2016/06/elasticsearch%e4%b9%8bshield%e8%ae%a4%e8%af%81%e4%b8%8e%e6%9d%83%e9%99%90%e7%ae%a1%e7%90%86.html'
views:
  - "26"
image: /wp-content/uploads/2016/06/shield-triad-220x150.png
categories:
  - 运维
tags:
  - ELK Stack
---
## 1.  Shield认证服务

Shield是Elastic公司为ElasticSearch开发的一个安全插件。在安装此插件后，Shield会拦截所有对ElasticSearch的请求，并加上认证与加密，保障ElasticSearch及相关系统的安全性。

## 2.支持功能

### 用户认证

使用Shield可以定义一系列已知的用户，并用其认证用户请求。这些用户存在于抽象的“域”中。一个域可能是下面几种类型：

### LDAP服务

Active Directory服务，本地esusers配置文件（类似/etc/passwd)

### 权限控制

Shield的权限控制包含下面几种元素：

  * 被保护的资源Secured Resource：权限所应用到的对象，比如某个index，cluster等等
  * 特权Priviliege：角色对对象可以执行的一种或多种操作，比如read,write等。还可以是indicies:/data/read/perlocate等某种对象特有的操作。
  * 许可Permissions：对被保护的资源拥有的一个或多个特权，如read on the &#8220;products&#8221; index.
  * 角色Role：由许可组成的有名字的集合.
  * 用户Users：用户实体，可以被赋予0种，1种或多种角色，他们可以对被保护的资源执行相应角色所拥有的各种特权。

### 集群节点认证与信道加密

Shield使用SSL/TLS加密相应端口(9300)，防止集群被未授权的机器监听或干扰。

### IP 过滤

Shield支持基于IP的访问控制。

### 审计

Shield可以在ElasticSearch的日志中输出每次鉴权操作的详细信息，包括用户名，操作，操作是否被允许等等。
  
Shield是商业插件，需要ElasticSearch的商业许可。第一次安装许可的时候，会提供30天的免费试用权限。30天后，Shield将会屏蔽cluster health, cluster stats, index stats这几个API，其余功能不受影响。

## 3.shield安装

进入elasticsearch安装目录中的bin,使用plugin命令执行插件的安装

<pre class="prettyprint linenums">$ /usr/share/elasticsearch/bin/plugin install elasticsearch/license/latest
$ /usr/share/elasticsearch/bin/plugin install elasticsearch/shield/latest
$ service elasticsearch restart
</pre>

如果采用deb或rpm方式安装，shield配置文件会写入/etc/elasticsearch/shield文件夹中（或者/usr/share/elasticsearch/config/shield）。

## 4.配置基本认证

基本身份认证，需要在创建用户时赋予不同的角色，shield角色默认包括3类：

  * admin:操作和管理集群和索引
  * power_user:监控集群和索引
  * user：所有索引的只读权限
  * transport_client
  * kibana4
  * kibana4_server
  * logstash
  * marvel_user

<pre class="prettyprint linenums">#####创建管理员账户
#添加用户es_admin，并加入admin角色中，密码必须6位以上
$ /usr/share/elasticsearch/bin/shield/esusers useradd es_admin -r admin

#测试使用es_admin用户访问elasticsearch
$ curl -u es_admin -XGET 'http://localhost:9200/'

###创建logstash角色用户
$ /usr/share/elasticsearch/bin/shield/esusers useradd logstashserver -r logstash

###创建Kibana角色用户
$ /usr/share/elasticsearch/bin/shield/esusers useradd kibanaserver -r kibana4_server
</pre>

一般不建议使用Shield，由于是商业软件安装成功有一定时间的试用期，导致之后部分功能将无法使用。

[<img class="aligncenter size-full wp-image-569" src="http://images.wanglijie.cn/public/img/posts/2016/06/shield-triad.png" alt="shield-triad" width="532" height="300" srcset="http://images.wanglijie.cn/public/img/posts/2016/06/shield-triad.png 532w, http://images.wanglijie.cn/public/img/posts/2016/06/shield-triad-300x169.png 300w" sizes="(max-width: 532px) 100vw, 532px" />](http://images.wanglijie.cn/public/img/posts/2016/06/shield-triad.png)

转载请注明：[自动化运维](http://www.wanglijie.cn) &raquo; [Elasticsearch之Shield认证与权限管理](http://www.wanglijie.cn/2016/06/elasticsearch%e4%b9%8bshield%e8%ae%a4%e8%af%81%e4%b8%8e%e6%9d%83%e9%99%90%e7%ae%a1%e7%90%86.html)