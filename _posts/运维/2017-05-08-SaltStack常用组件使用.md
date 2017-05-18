---
title: SaltStack常用组件使用
date: 2017-05-08T16:11:52+00:00
author: 深海游鱼
layout: post
permalink: '/2017/05/SaltStack常用组件使用.html'
image: 
categories:
  - 运维
tags:
  - SaltStack
  - 运维
---


# SaltStack常用组件

## 一、关于SaltStack的模块

SaltStack中的Module是我们日常使用SaltStack接触最多的一个组件，是用于管理对象操作的，这也是SaltStack通过Push的方式进行管理的入口，比如我们日常简单的执行命令、查看包安装情况、查看服务运行情况等工作都是通过SaltStack Module来实现的。默认安装好Master和Minion包之后，系统上会安装很多Module，大家可以通过以下命令查看支持的所有Module列表。

```
[root@pactera-jenkins-server pillar]# salt 'centos68-2*' sys.list_modules
centos68-2.test.pactera.top:
    - acl
    - aliases
    - alternatives
    - archive
    - artifactory
    - beacons
    - bigip
    - blockdev
    - bridge
    - btrfs
    - buildout
    - cloud
    - cmd
    - composer
    - config
    - consul
....
```

如果我们需要查看模块中具体可以使用的操作方法，则使用sys.list_functions
```
[root@pactera-jenkins-server pillar]# salt 'centos68-2*' sys.list_functions service
centos68-2.test.pactera.top:
    - service.available
    - service.delete
    - service.disable
    - service.disabled
    - service.enable
    - service.enabled
    - service.get_all
    - service.get_disabled
    - service.get_enabled
    - service.missing
    - service.reload
    - service.restart
    - service.start
    - service.status
    - service.stop
```
如果我们需要查看该模块的详细用法可以使用salt 'target' sys.doc [模块名称]



## 二、SaltStack 目标对象管理

在大规模服务器集群中，首先我们对维护好管理的对象，在SaltStack系统中把管理的目标对象称为”Target“，在Master上我们可以采用不同的Target去管理不同的Minion。这个Target通过管理和匹配Minion的ID来做一些集合的匹配。SaltStack管理对象拥有多种方式，包括Glob(默认)、PCRE、list、subnet、Grain、Grain PCRE、Pillar、Compound(混合)、Nodegroup(节点组).

### 2.1 目标对象匹配模式

1. 正则匹配
在操作与管理Minion时可以通过正则表达式来匹配Minion ID的方式去管理它们。比如我们想要对匹配到'Min*'字符串的Minion进行操作，可以按如下代码配置：
```
SaltStack@Master: salt -E  'Min*' test.ping 
Minion:
    True 
Minion1:
    True
Min*就是一个简单的正则表达式，当然你也可以写出任何正则表达式去匹配Minion ID。
```

2. 列表匹配
```
SaltStack@Master: salt -L  Minion,Minion1 test.ping
Minion:
    True 
Minion1:
    True

```
Minion和Minion1通过列表的方式去指定Minion ID，可直接使用。

3. Grians匹配
```
SaltStack@Master: salt -G  'os:MacOS' test.ping 
Minion:
    True 
Minion1:
    True
```

其中os:MacOS，这里的对象是一组键值对，这里用到了Minion的Grains的键值对。在后面介绍Grains的时候会详细讲解，这里只需要知道可以通过键值对的方式去匹配Minion ID。当然SaltStack也支持正则匹配Grains信息，大家可以通过--grain-pcre参数去匹配。

4. 组匹配
```
SaltStack@Master: salt -N  groups  test.ping 
Minion:
    True 
Minion1:
    True
```
在SaltStack系统中也可以提前给Minion定义组角色，但是需要提前知道Minion ID信息才能把它定义到某个组中。groups是我们在master配置文件中定义的组名称。
```
nodegroups:
    groups: 'L@Minon,Minion1
```

5. 复合匹配
```
SaltStack@Master: salt -C  'G@os:MacOS or L@Minion1'  test.ping 
Minion:
    True 
Minion1:
    True
```
os:MacOS or L@Minion1是一个复合组合，支持使用and和or关联多个条件。

6. Pillar值匹配
```
SaltStack@Master: salt -I  'key:value'  test.ping 
Minion:
    True 
Minion1:
    True
```

key:value是Pillar系统中定义的一组键值对，跟Grains的键值对类似。在下面的章节里面我们也会详细介绍SaltStack中的Pillar系统。

7. subnet匹配
```
SaltStack@Master: salt -S  '192.168.1.0/24'  test.ping 
Minion:
    True 
Minion1:
    True
```

192.168.1.0/24是一个指定的CIDR网段，这里CIDR匹配的IP地址是Minion连接Matser 4505端口的来源地址。

8. 常见的Target匹配模式如下表所示：

参数| 匹配模式        |       例子                           | 备注
  L | List of minions | L@Minion,Minion1,Minion2,Minion3     |
  G | Grains glob     | G@os:Ubuntu                          |操作系统平台不区分大小写
  E | PCRE minion ID  | E@Minion[1-3]                        |
  P | Grains PCRE     | P@os:(Centos\Fedora\Redhat)          |
  I | Pillar glob     | I@key:value                          |
  S | subnet/IP address | S@192.168.0.0/24 or S@192.168.0.122|
  R | Range cluster   | R@%foo.bar                           |
  C | compound        | G@os:MacOS or L@Minion1              |

### 2.2 管理对象属性

Grains是SaltStack组件中非常重要的组件之一，因为我们在做配置部署的过程中会经常使用它，Grains是SaltStack记录Minion的一些静态信息的组件，我们可以简单地理解为Grains里面记录着每台Minion的一些常用属性，比如CPU、内存、磁盘、网络信息等，我们可以通过grains.items查看某台Minion的所有Grains信息，Minions的Grains信息是Minions启动的时候采集汇报给Master的，在实际应用环境中我们需要根据自己的业务需求去自定义一些Grains，关于自定义Grains的常用方法有以下几种：
- 通过Minion配置文件定义。
- 通过Grains相关模块定义。
- 通过Python脚本定义。
```
SaltStack@Master: salt 'Minion' sys.list_functions grains
Minion:
    - grains.append
    - grains.delval
    - grains.filter_by
    - grains.get
    - grains.get_or_set_hash
    - grains.has_value
    - grains.item
    - grains.items
    - grains.ls
    - grains.remove
    - grains.setval
    - grains.setvals
```

#### 2.2.1 Minion配置文件定义Grains
下面我们先介绍下比较简单的Grains自定义方法，就是通过Minion配置文件定义。前面已经讲到Minions的Grains信息是在Minions服务启动的时候汇报给Matser的，所以我们需要修改好Minion配置文件后重启Minion服务。在Minion的/etc/salt/minion配置文件中默认有一些注释行。这里就是在Minion上的minion配置文件中如何定义Grains信息例子。下面只需根据自动的需求按照以下格式去填写相应的键值对就行，大家注意格式就行，SaltStack的配置文件的默认格式都是YAML格式：
```
#grains:
#  roles:
#    - webserver
#    - memcache
#  deployment: datacenter4
#  cabinet: 13
#  cab_u: 14-15
```

为了统一管理Minion的Grains信息，需要把这些注释复制到minion.d/grains文件中：
```
SaltStack@Minion: cat /etc/minion.d/grains
grains:
    roles:
        - webserver
        - memcache
deployment: datacenter4
    cabinet: 13
    cab_u: 14-15
```

然后通过/etc/init.d/salt-minion restart命令重启Minion服务。之后就可以去Master上查看定义的Grains信息是否生效：
```
SaltStack@Master: salt 'Minion' grains.item roles
Minion:
    ----------
    roles:
        - webserver
        - memcache 
SaltStack@Master: salt 'Minion' grains.item deployment
Minion:
    ----------
    cabinet:
        13
```

#### 2.2.2 Grains模块定义Grains

下面通过Grains模块定义Grains信息：
```
SaltStack@Master: salt 'Minion' grains.append saltbook 'verycool' #设置grains信息
Minion:
    ----------
    saltbook:
        - verycool
SaltStack@Master: salt 'Minion' grains.item saltbook #查看grains信息
Minion:
    ----------
    saltbook:
        - verycool
```

可以通过使用grains.setvals同时设置多对Grains信息：
```
[root@pactera-jenkins-server ~]# salt -E 'centos68-2.test.pactera.top' grains.setvals "{'salt':'good','book':'cool'}"
centos68-2.test.pactera.top:
    ----------
    book:
        cool
    salt:
        good
```

## 三、Pillar数据管理中心

### 3.1 什么是Pillar？

Pillar是SaltStack重要的组件之一，我们经常配合states在大规模配置管理工作中使用它，Pillar在SaltStack中主要作用就是存储和定义配置管理中需要的一些数据，比如版本号，用户名和密码等信息，它的定义存储格式跟Grains类似，都是YAML格式。在SaltStack Master配置文件中有一段 Pillar Settings选项定义了Pillar相关的参数：
```
#pillar_roots:
#    base:
#        - /srv/pillar
```

现在我们只需要了解pillar_roots相关的配置即可，默认Base环境下Pillar的工作目录在/srv/pillar目录下。如果你想定义多个环境不同的Pillar工作目录，只需要修改这处配置文件即可。下面我就用默认的配置，首先去pillar工作目录新建top.sls文件然后引用两个sls文件：
```
[root@pactera-jenkins-server pillar]# cat /srv/pillar/top.sls 
base:
    '*':
        - packages
        - services

[root@pactera-jenkins-server pillar]# cat /srv/pillar/packages.sls 
zabbix:
      package-name: zabbix
      version: 3.0.8
[root@pactera-jenkins-server pillar]# cat /srv/pillar/services.sls 
zabbix:
      port: 100050
      user: admin
```

通过以下命令查看关于Pillar相关的一些模块用法：

```
[root@pactera-jenkins-server pillar]# salt 'centos68-2*' sys.list_functions pillar
centos68-2.test.pactera.top:
    - pillar.data
    - pillar.ext
    - pillar.fetch
    - pillar.file_exists
    - pillar.get
    - pillar.item
    - pillar.items
    - pillar.keys
    - pillar.ls
    - pillar.obfuscate
    - pillar.raw
```

下面看一下刚刚定义的pillar
```
[root@pactera-jenkins-server pillar]# salt 'centos68-2*' pillar.item zabbix
centos68-2.test.pactera.top:
    ----------
    zabbix:
        ----------
        package-name:
            zabbix
        port:
            10050
        user:
            admin
        version:
            3.0.8
```

这个时候我们就可以查看到刚才定义的Pillar值。当然SaltStack也支持从外部读取Pillar数据。我们可以把Pillar数据存在数据库或者存储服务器上。目前官网也已经自带24种ext_pillar数据源了。

## 四、States配置管理

### 4.1 关于States
States是SaltStack中的配置语言，在日常进行配置管理时需要编写大量的States文件。比如我们需要安装一个包，然后管理一个配置文件，最后保证某个服务正常运行。这里就需要我们编写一些states sls文件（描述状态配置的文件）去描述和实现我们的功能。这里需要说明的是编写的states sls文件都是YAML语法。当然states sls文件也支持使用Python语言来编写。

### 4.2 States的用法

在大规模配置管理工作中，我们需要编写大量的states.sls文件。top.sls是states配置系统的入口文件，它在大规模配置管理工作中负责指定哪些设备调用



