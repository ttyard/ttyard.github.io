---
id: 546
title: ELK 综合日志归档分析系统(1)-Elasticsearch-Redis安装配置
date: 2016-06-29T17:58:54+00:00
author: 深海游鱼
layout: post
guid: http://www.wanglijie.cn/?p=546
permalink: '/2016/06/elk-%e7%bb%bc%e5%90%88%e6%97%a5%e5%bf%97%e5%bd%92%e6%a1%a3%e5%88%86%e6%9e%90%e7%b3%bb%e7%bb%9f1-elasticsearch%e5%ae%89%e8%a3%85%e9%85%8d%e7%bd%ae.html'
views:
  - "32"
image: /wp-content/uploads/2016/06/ELK-Mapping-220x150.png
categories:
  - 运维
tags:
  - elk
---
ELK全称（elasticsearch、logstash、kibana），是目前广泛使用的日志聚合和分析系统。该系统综合应用了这3种开源系统，构建日志采集传输（logstash）、存储搜索（elasticsearch）和前端展示分析（kibana）。

本文将将以ELK为平台,构建日志存储/分析系统，基于互联网的分享精神，转载请注明来源，谢谢。

## 1. ELK架构设计

[<img class="aligncenter size-full wp-image-566" src="http://images.wanglijie.cn/public/img/posts/2016/06/ELK架构图.jpg" alt="ELK架构图" width="653" height="619" srcset="http://images.wanglijie.cn/public/img/posts/2016/06/ELK架构图.jpg 653w, http://images.wanglijie.cn/public/img/posts/2016/06/ELK架构图-300x284.jpg 300w" sizes="(max-width: 653px) 100vw, 653px" />](http://images.wanglijie.cn/public/img/posts/2016/06/ELK架构图.jpg)

## 2. 安装Java运行环境

###  下载Java JRE

首先从Oracle官方网站下载最新的Java JDK或者JRE。

<pre class="prettyprint linenums">$ cd /opt
$ sudo wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" \
http://download.oracle.com/otn-pub/java/jdk/8u40-b25/jre-8u40-linux-x64.tar.gz
$ sudo tar xvf jre-8*.tar.gz
$ sudo chown -R root: jre1.8*
</pre>

###  安装Java JRE，配置环境变量

解压下载的tar.gz的安装包后，将Java环境变量加入到系统中，可以通过alternatives自动添加。

<pre class="prettyprint linenums">sudo alternatives --install /usr/bin/java java /opt/jre1.8*/bin/java 1
</pre>

或者也可以通过配置环境变量的方式来配置JRE（/etc/profile）

<pre class="prettyprint linenums">$ vim /etc/profile
JAVA_HOME=/usr/java/default
PATH=$JAVA_HOME/bin:$PATH
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export JAVA_HOME
export CLASSPATH
export PATH
</pre>

### 清理下载的安装文件，删除压缩包

<pre class="prettyprint linenums">$ sudo rm /opt/jre-8*.tar.gz
</pre>

## 3. Elasticsearch安装配置

### 3.1.  Elasticsearch简介

Elasticsearch是一个基于Apache Lucene(TM)的开源搜索引擎。无论在开源还是专有领域，Lucene可以被认为是迄今为止最先进、性能最好的、功能最全的搜索引擎库。

但是，Lucene只是一个库。想要使用它，你必须使用Java来作为开发语言并将其直接集成到你的应用中，更糟糕的是，Lucene非常复杂，

你需要深入了解检索的相关知识来理解它是如何工作的。

Elasticsearch也使用Java开发并使用Lucene作为其核心来实现所有索引和搜索的功能，但是它的目的是通过简单的RESTful API来隐藏Lucene的复杂性，从而让全文搜索变得简单。

不过，Elasticsearch不仅仅是Lucene和全文搜索，我们还能这样去描述它：

  * 分布式的实时文件存储，每个字段都被索引并可被搜索
  * 分布式的实时分析搜索引擎
  * 可以扩展到上百台服务器，处理PB级结构化或非结构化数据

而且，所有的这些功能被集成到一个服务里面，你的应用可以通过简单的RESTful API、各种语言的客户端甚至命令行与之交互。
  
上手Elasticsearch非常容易。它提供了许多合理的缺省值，并对初学者隐藏了复杂的搜索引擎理论。它开箱即用（安装即可使用），只需很少的学习既可在生产环境中使用。

Elasticsearch在Apache 2 license下许可使用，可以免费下载、使用和修改。
  
随着你对Elasticsearch的理解加深，你可以根据不同的问题领域定制Elasticsearch的高级特性，这一切都是可配置的，并且配置非常灵活。

### 3.2.  安装Elasticsearch

Elasticsearch支持源码包和在线仓库安装，由于海外站点速度较慢，推荐下载安装包或tar包后离线

**APT/Ubuntu**

<pre class="prettyprint linenums">#添加GPG-KEY
$ wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
#配置APT源
$ echo "deb http://packages.elastic.co/elasticsearch/2.x/debian stable main" | sudo tee -a /etc/apt/sources.list.d/elasticsearch-2.x.list
#安装elasticsearch
$ sudo apt-get update && sudo apt-get install elasticsearch
#配置Elasticsearch开机自动启动,将elasticsearch添加到 SysV init中.
$ sudo update-rc.d elasticsearch defaults 95 10</pre>

**RedHat/CentOS（YUM）**

<pre class="prettyprint linenums">#导入GPG-KEY
$ rpm --import https://packages.elastic.co/GPG-KEY-elasticsearch

#新增yum安装源
$ vim /etc/yum.repos.d/elasticsearch-2.repo
[elasticsearch-2.x]
name=Elasticsearch repository for 2.x packages
baseurl=https://packages.elastic.co/elasticsearch/2.x/centos
gpgcheck=1
gpgkey=https://packages.elastic.co/GPG-KEY-elasticsearch
enabled=1

#安装elasticsearch
$ yum install elasticsearch

#配置开机自动启动
#CentOS6 
chkconfig --add elasticsearch 
chkconfig elasticsearch on 
service elasticsearch start 
#CentOS7 
sudo systemctl start elasticsearch.service 
sudo systemctl enable elasticsearch.service
</pre>

&nbsp;

### 3.3.  配置Elasticsearch

Elasticsearch默认的配置文件位于/etc/elasticsearch/elasticsearch.yml，安装下面的参数说明，配置和修改相应的参数。

创建elasticsearch数据存储目录

<pre class="prettyprint linenums">mkdir -p /data/elasticsearch/logs
mkdir -p /data/elasticsearch/data
chown -R elasticsearch:elasticsearch /data/elasticsearch
chmod -R ug+rx /data/elasticsearch
</pre>

### 3.3.1 基础配置修改

### 配置elasticsearch参数

<pre class="prettyprint linenums">vim /etc/elasticsearch/elasticsearch.yml
</pre>

  * 修改集群name 为 edselk
  * 修改节点名称name 为 &#8220;edselk-node1&#8221;
  * 修改elasticsearch数据存储位置data 为 /data/elasticsearch/data
  * 修改elasticsearch日志存储位置logs 为 /data/elasticsearch/logs
  * 限制外部访问elasticsearch的9200端口，不允许从外部通过HTTP AIP关闭elasticsearch集群和elasticsearch的数据。
  * 找到host，修改成localhost
  * 最后确认修改的配置内容

<pre class="prettyprint linenums">$ cat /etc/elasticsearch/elasticsearch.yml |grep -v "^#"|grep -v "^$"
cluster.name: edselk
node.name: "edselk-node1"
path.data: /data/elasticsearch/data
path.logs: /data/elasticsearch/logs
network.host: localhost
transport.tcp.port: 9300
http.port: 9200
</pre>

### 3.3.2 集群配置

elasticsearch集群自动发现采用P2P(gossip协议)，除了集群状态管理外，其他所有请求都可以发送到集群内的任意节点上，节点可以自己找到需要转发给那个阶段，并且直接跟这些节点进行通讯。

在无阻碍网络，只要配置了相同cluster.name的节点都会自动归集到一个集群中，此时采用的是muliticast（组播）方式，使用组播地址224.2.2.4，以54328端口建立组播组发送cluster.name信息，但一般网络管理员会禁用组播在网路中的传播。

**head集群管理插件**

<pre class="prettyprint linenums">#Ubuntu Linux
$ sudo elasticsearch/bin/plugin install mobz/elasticsearch-head

#Windows
c:\Server\elasticsearch-2.1.1\bin&gt;plugin.bat install mobz/elasticsearch-head
-&gt; Installing mobz/elasticsearch-head...
Trying https://github.com/mobz/elasticsearch-head/archive/master.zip ...
Downloading ..................................................DONE
Verifying https://github.com/mobz/elasticsearch-head/archive/master.zip checksums if available ...
NOTE: Unable to verify checksum for downloaded plugin (unable to find .sha1 or .md5 file to verify)
Installed head into c:\Server\elasticsearch-2.1.1\plugins\head
</pre>

安装完毕后，在浏览器地址栏输入http://ip:9200/_plugin/head/ 即可打开head集群管理工具，如下图所示：
  
[<img class="aligncenter wp-image-579" src="http://images.wanglijie.cn/public/img/posts/2016/06/elastic-head-1.png" alt="elastic-head" width="790" height="562" />](http://images.wanglijie.cn/public/img/posts/2016/06/elastic-head-1.png)

### 3.3.3 安全配置

依据不同的安全项目，修改elasticsearch.yml中的配置项目。默认配置是打开动态脚本功能的，如果用户没有更改默认配置文件，攻击者可以直接通过http请求执行任意代码

<pre class="prettyprint linenums">#禁用动态脚本
script.disable_dynamic: true
#Groovy引擎启用沙盒模式
script.groovy.sandbox.enabled: true
#任意文件读取漏洞
#使用本地网络启动elasticsearch监听地址
network.bind_host:10.12.49.169
</pre>

### 3.3.4 配置文件详解

<pre class="prettyprint linenums">### /etc/elasticsearch/elasticsearch.yml
cluster.name: elasticsearch
配置es的集群名称，默认是elasticsearch，es会自动发现在同一网段下的es，如果在同一网段下有多个集群，就可以用这个属性来区分不同的集群。

node.name: “Franz Kafka”
节点名，默认随机指定一个name列表中名字，该列表在es的jar包中config文件夹里name.txt文件中，其中有很多作者添加的有趣名字。

node.master: true
指定该节点是否有资格被选举成为node，默认是true，es是默认集群中的第一台机器为master，如果这台机挂了就会重新选举master。

node.data: true
指定该节点是否存储索引数据，默认为true。

index.number_of_shards: 5
设置默认索引分片个数，默认为5片。

index.number_of_replicas: 1
设置默认索引副本个数，默认为1个副本。

path.conf: /path/to/conf
设置配置文件的存储路径，默认是es根目录下的config文件夹。

path.data: /path/to/data
设置索引数据的存储路径，默认是es根目录下的data文件夹，可以设置多个存储路径，用逗号隔开，例：

path.data: /path/to/data1,/path/to/data2
path.work: /path/to/work
设置临时文件的存储路径，默认是es根目录下的work文件夹。

path.logs: /path/to/logs
设置日志文件的存储路径，默认是es根目录下的logs文件夹

path.plugins: /path/to/plugins
设置插件的存放路径，默认是es根目录下的plugins文件夹

bootstrap.mlockall: true
设置为true来锁住内存。因为当jvm开始swapping时es的效率会降低，所以要保证它不swap，可以把ES_MIN_MEM和ES_MAX_MEM两个环境变量设置成同一个值，并且保证机器有足够的内存分配给es。同时也要允许elasticsearch的进程可以锁住内存，linux下可以通过`ulimit -l unlimited`命令。

network.bind_host: 10.112.49.169
设置绑定的ip地址，可以是ipv4或ipv6的，默认为0.0.0.0。

network.publish_host: 10.112.49.169
设置其它节点和该节点交互的ip地址，如果不设置它会自动判断，值必须是个真实的ip地址。

network.host: 10.112.49.169
这个参数是用来同时设置bind_host和publish_host上面两个参数。

transport.tcp.port: 9300
设置节点间交互的tcp端口，默认是9300。

transport.tcp.compress: true
设置是否压缩tcp传输时的数据，默认为false，不压缩。

http.port: 9200
设置对外服务的http端口，默认为9200。

http.max_content_length: 100mb
设置内容的最大容量，默认100mb

http.enabled: false
是否使用http协议对外提供服务，默认为true，开启。

gateway.type: local
gateway的类型，默认为local即为本地文件系统，可以设置为本地文件系统，分布式文件系统，hadoop的HDFS，和amazon的s3服务器，其它文件系统的设置方法下次再详细说。

gateway.recover_after_nodes: 1
设置集群中N个节点启动时进行数据恢复，默认为1。

gateway.recover_after_time: 5m
设置初始化数据恢复进程的超时时间，默认是5分钟。

gateway.expected_nodes: 2
设置这个集群中节点的数量，默认为2，一旦这N个节点启动，就会立即进行数据恢复。

cluster.routing.allocation.node_initial_primaries_recoveries: 4
初始化数据恢复时，并发恢复线程的个数，默认为4。

cluster.routing.allocation.node_concurrent_recoveries: 2
添加删除节点或负载均衡时并发恢复线程的个数，默认为4。

indices.recovery.max_size_per_sec: 0
设置数据恢复时限制的带宽，如入100mb，默认为0，即无限制。

indices.recovery.concurrent_streams: 5
设置这个参数来限制从其它分片恢复数据时最大同时打开并发流的个数，默认为5。

discovery.zen.minimum_master_nodes: 1
设置这个参数来保证集群中的节点可以知道其它N个有master资格的节点。默认为1，对于大的集群来说，可以设置大一点的值（2-4）

discovery.zen.ping.timeout: 3s
设置集群中自动发现其它节点时ping连接超时时间，默认为3秒，对于比较差的网络环境可以高点的值来防止自动发现时出错。

discovery.zen.ping.multicast.enabled: false
设置是否打开多播发现节点，默认是true。

discovery.zen.ping.unicast.hosts: ["host1", "host2:port", "host3[portX-portY]“]
设置集群中master节点的初始列表，可以通过这些节点来自动发现新加入集群的节点。

下面是一些查询时的慢日志参数设置
index.search.slowlog.level: TRACE
index.search.slowlog.threshold.query.warn: 10s
index.search.slowlog.threshold.query.info: 5s
index.search.slowlog.threshold.query.debug: 2s
index.search.slowlog.threshold.query.trace: 500ms

index.search.slowlog.threshold.fetch.warn: 1s
index.search.slowlog.threshold.fetch.info: 800ms
index.search.slowlog.threshold.fetch.debug:500ms
index.search.slowlog.threshold.fetch.trace: 200ms
</pre>

### 

## 3. Redis 配置管理

### 3.1.  Redis简介

Redis是一个开源的使用ANSI C语言编写、支持网络、可基于内存亦可持久化的日志型、Key-Value数据库，并提供多种语言的API。从2010年3月15日起，Redis的开发工作由VMware主持。

  * **数据模型**：作为Key-value型数据库，Redis也提供了键（Key）和键值（Value）的映射关系。但是，除了常规的数值或字符串，Redis的键值还可以是以下形式之一：(1)Lists （列表）
  
    (2)Sets （集合）
  
    (3)Sorted sets （有序集合）
  * **Hashes （哈希表）**：键值的数据类型决定了该键值支持的操作。Redis支持诸如列表、集合或有序集合的交集、并集、差集等高级原子操作；同时，如果键值的类型是普通数字，Redis则提供自增等原子操作。
  * **持久化**：通常，Redis将数据存储于内存中，或被配置为使用虚拟内存。通过两种方式可以实现数据持久化：使用快照的方式，将内存中的数据不断写入磁盘；或使用类似MySQL的日志方式，记录每次更新的日志。前者性能较高，但是可能会引起一定程度的数据丢失；后者相反。
  * **主从同步**：Redis支持将数据同步到多台从库上，这种特性对提高读取性能非常有益。
  * **性能**：相比需要依赖磁盘记录每个更新的数据库，基于内存的特性无疑给Redis带来了非常优秀的性能。读写操作之间没有显著的性能差异，如果Redis将数据只存储于内存中。

### Redis 与memcached:

1.Redis中，并不是所有的数据都一直存储在内存中的，这是和Memcached相比一个最大的区别。
  
2.Redis不仅仅支持简单的k/v类型的数据，同时还提供list，set，hash等数据结构的存储。
  
3.Redis支持数据的备份，即master-slave模式的数据备份。
  
4.Redis支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用。

### 3.2.  安装redis

  * 在线安装

<pre class="prettyprint linenums">yum search redis
yum install redis
</pre>

  * 源码安装

<pre class="prettyprint linenums">wget <a href="http://download.redis.io/releases/redis-3.0.2.tar.gz">http://download.redis.io/releases/redis-3.0.2.tar.gz</a>
</pre>

解压下载的压缩包文件

<pre class="prettyprint linenums">tar redis-3.0.2.tar.gz
cd redis-3.0.2 && make
yum –y install tcl
cd src && make test
cp src/redis-server /usr/local/bin/
cp src/redis-cli /usr/local/bin/
mkdir /etc/redis
mkdir /var/redis
cp utils/redis_init_script /etc/init.d/redis
cp redis.conf /etc/redis/6379.conf
mkdir /var/redis/6379
mkdir /var/log/redis
</pre>

编辑配置文件，修改下面的内容

<pre class="prettyprint linenums">daemonize yes
pidfile /var/run/redis_6379.pid
appendonly yes
logfile "/var/log/redis/6379.log"
</pre>

启动redis服务

<pre class="prettyprint linenums">service redis start
#查看redis实时内容
redis-cli -p 6379 monitor
</pre>

### 3.3.  Redis安全配置

### 密码认证

Redis的配置文件默认在/etc/redis.conf，找到如下行：

<pre class="prettyprint linenums">#requirepass foobared
#去掉前面的注释，并修改为所需要的密码：
requirepass GZcY*****Vm
#重命名Redis CONFIG命令
rename-command CONFIG "N8****UE"
</pre>

### Spiped数据传输加密

Redis默认并不支持数据传输过程中的通讯加密，为了配置Redis在非信任的网络中访问，可以通过而外的安装错误保护数据传输的安全，如支持SSL Proxy的spiped。

  * **安装Spiped**

<pre class="prettyprint linenums">apt-get install libssl-dev
wget http://www.tarsnap.com/spiped/spiped-1.5.0.tgz
tar -xzvf spiped-1.5.0.tgz
cd spiped-1.5.0
make
make install
</pre>

  * 配置

服务器端建立公网IP的7480端口à本地6379。

<pre class="prettyprint linenums">#将生成的spiped.keyfile复制到目标机器
dd if=/dev/urandom bs=2048 count=1 of=/etc/pki/spiped.keyfile
#启动Redis加密通讯的端口
spiped -d -s '[0.0.0.0]:7480' -t '[127.0.0.1]:6379' -k /etc/pki/spiped.keyfile
</pre>

远程客户端通过服务器端建立的7480端口建立加密的socket连接，并在本地启动端口6379来进行数据通讯。

<pre class="prettyprint linenums">spiped -e -s '[127.0.0.1]:6379' -t $SERVERNAME:7480'  -k /etc/pki/spiped.keyfile
</pre>

至此，elasticsearch与Redis安装完毕，下篇将详细介绍logstash安装配置。

转载请注明：[自动化运维](http://www.wanglijie.cn) &raquo; [ELK 综合日志归档分析系统(1)-Elasticsearch-Redis安装配置](http://www.wanglijie.cn/2016/06/elk-%e7%bb%bc%e5%90%88%e6%97%a5%e5%bf%97%e5%bd%92%e6%a1%a3%e5%88%86%e6%9e%90%e7%b3%bb%e7%bb%9f1-elasticsearch%e5%ae%89%e8%a3%85%e9%85%8d%e7%bd%ae.html)