---
id: 484
title: SaltStack初始化安装配置
date: 2016-06-02T15:49:16+00:00
author: 深海游鱼
layout: post
guid: http://www.wanglijie.cn/?p=484
permalink: '/2016/06/saltstack%e5%88%9d%e5%a7%8b%e5%8c%96%e5%ae%89%e8%a3%85%e9%85%8d%e7%bd%ae.html'
views:
  - "44"
image: /wp-content/uploads/2016/06/saltstack-220x72.png
categories:
  - 运维
tags:
  - SaltStack
---
## 一、SaltStack简介

SaltStack是一个服务器基础架构集中化管理平台，salt底层采用动态的连接总线，具备配置管理、远程执行、监控等功能，她是一种全新的基础设施管理方式，部署轻松，在几分钟内可运行起来，扩展性好，速度够快，服务器之间秒级通讯。
  
一般可以将SaltStack理解为简化版的puppet和加强版的func。SaltStack基于Python语言实现，结合轻量级消息队列（ZeroMQ）与Python第三方模块（Pyzmq、PyCrypto、Pyjinjia2、python-msgpack和PyYAML等）构建。
  
通过部署SaltStack环境，我们可以在成千上万台服务器上做到批量执行命令，根据不同业务特性进行配置集中化管理、分发文件、采集服务器数据、操作系统基础及软件包管理等，SaltStack是运维人员提高工作效率、规范业务配置与操作的利器。

<img class="aligncenter size-full wp-image-485" src="http://images.wanglijie.cn/public/img/posts/2016/06/saltstack.png" alt="saltstack" width="308" height="72" srcset="http://images.wanglijie.cn/public/img/posts/2016/06/saltstack.png 308w, http://images.wanglijie.cn/public/img/posts/2016/06/saltstack-300x70.png 300w" sizes="(max-width: 308px) 100vw, 308px" />

## 二、saltstack安装

bootstrap-salt.sh是自动安装脚本，能够根据操作系统平台识别并安装相应的SaltStack版本。
  
`https://raw.githubusercontent.com/saltstack/salt-bootstrap/stable/bootstrap-salt.sh`

### 2.1 Ubuntu 14.04

最新版的SaltStack通过Ubuntu PPA进行发布，通过下面命令添加PPA

<pre class="prettyprint linenums">sudo add-apt-repository ppa:saltstack/salt
</pre>

安装salt-master、salt-minion或salt-syndic。

<pre class="prettyprint linenums">sudo apt-get update
sudo apt-get -y install salt-master
sudo apt-get -y install salt-minion
#salt-syndic 类似于代理接口
#sudo apt-get install salt-syndic
</pre>

### 2.2 RHEL/CENTOS 6 AND 7, SCIENTIFIC LINUX

SaltStack RPM安装包已包含在EPEL的库中，根据下面的版本下载并安装EPEL

<pre class="prettyprint linenums">http://download.fedoraproject.org/pub/epel/6/i386/repoview/epel-release.html
http://download.fedoraproject.org/pub/epel/6/x86_64/repoview/epel-release.html
http://download.fedoraproject.org/pub/epel/7/x86_64/repoview/epel-release.html
#CENTOS6 x86_64
rpm -ivh  http://mirrors.opencas.cn/epel/6/x86_64/epel-release-6-8.noarch.rpm
#CENTOS6 x86_32
rpm -ivh http://mirrors.opencas.cn/epel/6/i386/epel-release-6-8.noarch.rpm
#CENTOS7 x86_64
rpm -ivh http://mirrors.opencas.cn/epel/7/x86_64/e/epel-release-7-5.noarch.rpm
#安装SaltStack
yum install salt-master salt-minion
</pre>

安装完成后，默认SaltStack配置未见位于/etc/salt目录下，如下：

<pre class="prettyprint linenums">/etc/salt/
├── master
├── master.d
├── minion
├── minion.d
├── minion_id
└── pki
    ├── master
    │   ├── master.pem
    │   ├── master.pub
    │   ├── minions
    │   │   ├── VM-OptDevRHEL-49-35
    │   │   └── VM-Ubuntu1404-49-70
    │   ├── minions_pre
    │   └── minions_rejected
    └── minion
</pre>

### 2.3 配置防火墙

默认SaltStack master需要开放TCP 4505,4506端口外网通讯，salt-minion不需要开放端口，它能够主动与salt-master 4505/tcp连接通讯。

<pre class="prettyprint linenums">#CENTOS6
iptables -A INPUT -m state --state new -m tcp -p tcp --dport 4505 -j ACCEPT
iptables -A INPUT -m state --state new -m tcp -p tcp --dport 4506 -j ACCEPT
service iptables save
#CENTOS 7
firewall-cmd --permanent --zone=public --add-port=4505-4506/tcp
firewall-cmd --reload
#Ubuntu/Debain
ufw allow 4505/tcp
ufw allow 4506/tcp
</pre>

## 三、配置SaltStack Master、Minion

### 3.1 初始化salt-minion配置文件

Salt-master初始化配置基本上不需要修改什么参数。初步测试，只需要保持默认配置即可，详细配置在使用的时候在修改。
  
修改minion配置，使其能够连接到salt-master。

<pre class="prettyprint linenums">vim /etc/salt/minion
master: salt.example.com
id: 172-22-16-10-BJ-CN
</pre>

注意：
  
1.配置文件中冒号后面需要有“空格”。
  
2.如果不配置minion id，将使用主机hostname。

启动SaltStack服务

<pre class="prettyprint linenums">sudo service salt-master start
sudo service salt-minion start
</pre>

### 3.2 客户端密钥配置

salt-minion安装成功启动后，会在/etc/salt/pki/minions下创建RSA的公私钥，用于与Salt-Master进行通讯。我们需要在salt-master中配置并接受该机器和密钥的连接才能正常的进行管理。

<pre class="prettyprint linenums">salt-key 常用的命令选项 
-a [option] 确认指定的证书
-A          确认所有的证书
-d [option] 删除指定的证书
-D          删除所有的证书
-L          列出所有证书
</pre>

查看所有连接到master的minion密钥。该命令显示的是minion的id或主机名

<pre class="prettyprint linenums">root@ubuntu:~/salt# salt-key
Accepted Keys:
Unaccepted Keys:
VM-OptDevRHEL-49-35
VM-Ubuntu1404-49-70
Rejected Keys:
</pre>

查看密钥的公钥指纹

<pre class="prettyprint linenums">root@ubuntu:~/salt# salt-key -f VM-Ubuntu1404-49-70
Accepted Keys:
VM-Ubuntu1404-49-70:  fc:73:4d:0a:71:8a:a5:3b:b3:cd:7c:1c:d3:f3:75:ac
</pre>

在Salt-minion上查看KEY公钥指纹信息

<pre class="prettyprint linenums">root@ubuntu:~# salt-call --local key.finger
local:
    fc:73:4d:0a:71:8a:a5:3b:b3:cd:7c:1c:d3:f3:75:ac
</pre>

如果KEY公钥指纹信息匹配，可以通过下面命令接受该公钥

<pre class="prettyprint linenums">root@ubuntu:~# salt-call --local key.finger
local:
    fc:73:4d:0a:71:8a:a5:3b:b3:cd:7c:1c:d3:f3:75:ac
</pre>

如果接受所有KEY，可以使用salt-key -A

<pre class="prettyprint linenums">root@ubuntu:~/salt# salt-key -a VM-Ubuntu1404-49-70
The following keys are going to be accepted:
Unaccepted Keys:
VM-Ubuntu1404-49-70
Proceed? [n/Y] Y
Key for minion myminion accepted.
</pre>

注意：如果salt-minion连接的salt-master服务端密钥发生变更，这需要在/etc/salt/pki/minion/minion_master.pub处进行修改。

### 3.3 测试salt-minion 客户端

<pre class="prettyprint linenums">[root@ubuntu~]# salt '*' test.ping
localhost:
    True
Company-WIN08R2-SERV1:
    True
Company-Linux-Ubuntu-10-12-49-70:
    True
Personal-WIN2003R2-10-1-1-11:
    True
Company-Linux-Centos-10-12-49-35:
    Minion did not return. [No response]</pre>

### 注意：

saltstack-master在跨越防火墙部署时，有时候有因为zeroMQ问题导致通讯失败。

转载请注明：[自动化运维](http://www.wanglijie.cn) &raquo; [SaltStack初始化安装配置](http://www.wanglijie.cn/2016/06/saltstack%e5%88%9d%e5%a7%8b%e5%8c%96%e5%ae%89%e8%a3%85%e9%85%8d%e7%bd%ae.html)