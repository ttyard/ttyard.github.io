---
id: 421
title: 生成Linux PEM/CER格式SSH身份验证密钥 (Windows Azure)
date: 2015-12-08T21:09:25+00:00
author: 深海游鱼
layout: post
guid: http://www.wanglijie.cn/?p=421
permalink: '/2015/12/%e7%94%9f%e6%88%90linux-pemcer%e6%a0%bc%e5%bc%8fssh%e8%ba%ab%e4%bb%bd%e9%aa%8c%e8%af%81%e5%af%86%e9%92%a5-windows-azure.html'
views:
  - "254"
categories:
  - 云计算
tags:
  - Azure
  - OpenSSL
---
在Windows Azure上创建Linux虚拟机时，如果想采用SSH Key方式的证书密钥登录Linux 服务器，必须在创建虚拟机同时上传兼容的 SSH 密钥以进行身份，该密钥必须是符合X.509标准的cer或pem格式的密钥。

通过中文版帮助文档创建的x509证书在linux能正常使用，当时在Windows Xshell和Putty使用私钥时提示无法打开问题。可以通过下面的方法先用ssh生成公私钥，然后使用openssl进行证书格式转换。

## 1.使用ssh-keygen生成ssh证书

<pre class="prettyprint linenums">root@F3L:~# ssh-keygen 
#如果命令无效，需要首先安装openssl-client
使用命令查询具体名称安装包名称
#Ubuntu系统
apt-cache search openssl-client
#CENTOS/RedHat
yum search openssl-clients
</pre>

## 2.将证书转换成x509格式

<pre class="prettyprint linenums">openssl req -x509 -key ~/.ssh/id_rsa -nodes -days 365 -newkey rsa:2048 -out myCert.pem
</pre>

### 3.将.pem证书转换成cer格式

<pre class="prettyprint linenums">openssl x509 -outform der -in myCert.pem -out myCert.cer
</pre>

## 4.上传.pem或.cer公钥

SSH密钥创建完成后，在创建虚拟机界面上传密钥的地方将.pem或者.cer格式的私钥上传到Azure控制台即可。

[<img class="aligncenter size-full wp-image-424" src="http://images.wanglijie.cn/public/img/posts/2015/12/azure-ssh-key1.jpg" alt="azure-ssh-key" width="411" height="573" srcset="http://images.wanglijie.cn/public/img/posts/2015/12/azure-ssh-key1.jpg 411w, http://images.wanglijie.cn/public/img/posts/2015/12/azure-ssh-key1-215x300.jpg 215w" sizes="(max-width: 411px) 100vw, 411px" />](http://images.wanglijie.cn/public/img/posts/2015/12/azure-ssh-key1.jpg)

## 5.下载SSH私钥到本地系统

默认创建的SSH私钥文件路径是~/.ssh/id_rsa，将该文件下载到系统，直接导入Xshell中即可使用。
  
如果使用Putty，需要使用putty-keygen将私钥文件转换层.ppk文件即可使用顺利Linux登录系统。

Windows Azure官方参考文档：<a href="http://azure.microsoft.com/en-us/documentation/articles/virtual-machines-linux-use-ssh-key/?rnd=1" target="_blank">点击此处链接</a>

转载请注明：[自动化运维](http://www.wanglijie.cn) &raquo; [生成Linux PEM/CER格式SSH身份验证密钥 (Windows Azure)](http://www.wanglijie.cn/2015/12/%e7%94%9f%e6%88%90linux-pemcer%e6%a0%bc%e5%bc%8fssh%e8%ba%ab%e4%bb%bd%e9%aa%8c%e8%af%81%e5%af%86%e9%92%a5-windows-azure.html)