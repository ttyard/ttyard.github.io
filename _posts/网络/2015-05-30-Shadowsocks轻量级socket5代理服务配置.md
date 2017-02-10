---
id: 283
title: Shadowsocks轻量级socket5代理服务配置
date: 2015-05-30T13:06:28+00:00
author: 深海游鱼
layout: post
guid: http://www.wanglijie.cn/?p=283
permalink: '/2015/05/shadowsocks%e8%bd%bb%e9%87%8f%e7%ba%a7socket5%e4%bb%a3%e7%90%86%e6%9c%8d%e5%8a%a1%e9%85%8d%e7%bd%ae.html'
views:
  - "456"
image: /wp-content/uploads/2015/05/3006190-200x150.png
categories:
  - 网络
---
Shadowsocks是一个轻量级socks5代理，以python写成，安装方便，配置简单。它的配置文件是以json格式存在。下面将详细介绍安装配置方法。

## Shadowsocks服务端

为了Shadowsocks能够上国外网站，你需要有一台海外的服务器并能够正常上外网。

### 服务端安装

Debian / Ubuntu:

<pre class="prettyprint linenums">apt-get install python-pip
pip install shadowsocks
</pre>

CentOS:

<pre class="prettyprint linenums">yum install python-setuptools && easy_install pip
pip install shadowsocks
</pre>

安装成功后，默认的程序路径：/usr/local/bin/ssserver

### 生成配置文件

创建/etc/shadowsocks/server.json配置文件，根据你的需要修改默认的配置文件具体内容如下：

<pre class="prettyprint linenums">{
	"server":"0.0.0.0",
	"server_port":443,
	"local_address":"127.0.0.1",
	"local_port":1080,
	"password":"your-passwd",
	"timeout":300,
	"method":"aes-256-cfb",
	"fast_open":false,
	"workers":1
}
</pre>

提示: 若需同时指定多个服务端ip，可参考：

<pre class="prettyprint linenums">"server":["1.1.1.1","2.2.2.2"],
</pre>

server: 服务端监听地址(IPv4或IPv6)
  
server_port: 服务端端口，一般为443
  
local_address: 本地监听地址，缺省为127.0.0.1
  
local_port : 本地监听端口，一般为1080
  
password: 用以加密的密匙
  
timeout: 超时时间（秒）
  
method: 加密方法，默认的table是一种不安全的加密，此处首推aes-256-cfb
  
fast_open: 是否启用[TCP-Fast-Open](https://github.com/clowwindy/shadowsocks/wiki/TCP-Fast-Open)
  
wokers: worker数量，如果不理解含义请不要改

### 配置多种加密方法

**安装python-m2crypto**
  
Debian:

<pre class="prettyprint linenums">apt-get install python-m2crypto
</pre>

CentOS:

<pre class="prettyprint linenums">yum install m2crypto
</pre>

支持的加密算法：
  
|aes-256-cfb: Default  |aes-128-cfb|aes-192-cfb|aes-256-ofb|aes-128-ofb|
|aes-192-ofb|aes-128-ctr|aes-192-ctr|aes-256-ctr|aes-128-cfb8|
|aes-192-cfb8|aes-256-cfb8|aes-128-cfb1|aes-192-cfb1|aes-256-cfb1|
|bf-cfb|camellia-128-cfb|camellia-192-cfb|camellia-256-cfb|cast5-cfb|
|chacha20|idea-cfb|rc2-cfb|rc4-md5|salsa20
|seed-cfb|

**salsa20和chacha20加密算法**
  
salsa20和chacha20是非常快的加密算法，是rc4的两倍。默认并不支持，需要编译安装，如下：

<pre class="prettyprint linenums">apt-get install build-essential
wget https://github.com/jedisct1/libsodium/releases/download/1.0.1/libsodium-1.0.1.tar.gz
tar xf libsodium-1.0.1.tar.gz && cd libsodium-1.0.1
./configure && make -j2 && make install
ldconfig
</pre>

<a href="https://github.com/shadowsocks/shadowsocks/wiki/Encryption" target="_blank">详细参考：shadowsocks Encryption</a>

### 运行shadowsocks

/usr/local/bin/ssserver -c /etc/shadowsocks/server.json -d start

## 内核优化

<pre class="prettyprint linenums">vim /etc/sysctl.conf

net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 0
net.ipv4.tcp_fin_timeout = 30
net.ipv4.tcp_keepalive_time = 1200
net.ipv4.ip_local_port_range = 10000 65000
net.ipv4.tcp_max_syn_backlog = 8192
net.ipv4.tcp_max_tw_buckets = 5000
net.ipv4.tcp_fastopen = 3
net.ipv4.tcp_rmem = 4096 87380 67108864
net.ipv4.tcp_wmem = 4096 65536 67108864
net.ipv4.tcp_mtu_probing = 1
net.ipv4.tcp_congestion_control = hybla
#使配置文件生效
sysctl -p
</pre>

## 安装客户端

在config.json所在目录下运行sslocal即可；若需指定配置文件的位置：

<pre class="prettyprint linenums">
sslocal -c /etc/shadowsocks/config.json
</pre>

注意: 有用户报告无法成功在运行时加载config.json，或可尝试手动运行：

<pre class="prettyprint linenums">sslocal -s 服务器地址 -p 服务器端口 -l 本地端端口 -k 密码 -m 加密方法
</pre>

提示: 当然也有图形化的使用<a href="https://github.com/shadowsocks/shadowsocks-gui" target="_blank">shadowsocks-gui@gitHub</a>,如果不希望自己编译的话，也可以到<a href="http://sourceforge.net/projects/shadowsocksgui/files/dist/" target="_blank">shadowsocks-gui@sourceforge</a>下载。
  
如果上面的下载连接无法访问，从此处下载<a href="http://www-aixiuyun.qiniudn.com/Shadowsocks.zip" target="_blank">Windows shadowsocks</a>
  
[<img class="aligncenter size-full wp-image-332" src="http://images.wanglijie.cn/public/img/posts/2015/05/QQ拼音截图未命名.jpg" alt="QQ拼音截图未命名" width="281" height="307" srcset="http://images.wanglijie.cn/public/img/posts/2015/05/QQ拼音截图未命名.jpg 281w, http://images.wanglijie.cn/public/img/posts/2015/05/QQ拼音截图未命名-275x300.jpg 275w" sizes="(max-width: 281px) 100vw, 281px" />](http://images.wanglijie.cn/public/img/posts/2015/05/QQ拼音截图未命名.jpg)

参考连接：<a href="https://wiki.archlinux.org/index.php/Shadowsocks_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)" target="_blank">archlinux</a>

转载请注明：[自动化运维](http://www.wanglijie.cn) &raquo; [Shadowsocks轻量级socket5代理服务配置](http://www.wanglijie.cn/2015/05/shadowsocks%e8%bd%bb%e9%87%8f%e7%ba%a7socket5%e4%bb%a3%e7%90%86%e6%9c%8d%e5%8a%a1%e9%85%8d%e7%bd%ae.html)