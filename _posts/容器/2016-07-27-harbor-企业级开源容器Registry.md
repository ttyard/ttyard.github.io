---
id: 638
title: Harbor 企业级开源容器Registry
date: 2016-07-27T09:40:01+00:00
author: 深海游鱼
layout: post
guid: http://www.wanglijie.cn/?p=638
permalink: '/2016/07/harbor-%e4%bc%81%e4%b8%9a%e7%ba%a7%e5%bc%80%e6%ba%90%e5%ae%b9%e5%99%a8registry.html'
views:
  - "12"
image: /wp-content/uploads/2016/07/harbor-logo-220x150.png
categories:
  - 容器
tags:
  - Docker
---
## **关于Harbor项目**

VMware公司3月份开源了企业级Registry项目Harbor，由VMware中国研发的团队负责开发。Harbor项目是帮助用户迅速搭建一个企业级的registry 服务。它以Docker公司开源的registry为基础，提供了管理图形界面, 基于角色的访问控制(Role Based Access Control)，镜像远程复制（同步），AD/LDAP集成、以及审计日志(Audit logging) 等企业用户需求的功能，同时还原生支持中文，对广大中国用户是一个好消息。该项目推出4个月以来，在GitHub 获得了超过800个点赞的星星和200多个 forks。本文将介绍Harbor项目的主要组件，并阐述Harbor的工作原理。

<img class="aligncenter size-full wp-image-643" src="http://images.wanglijie.cn/public/img/posts/2016/07/harbor-logo.png" alt="harbor-logo" width="350" height="163" srcset="http://images.wanglijie.cn/public/img/posts/2016/07/harbor-logo.png 350w, http://images.wanglijie.cn/public/img/posts/2016/07/harbor-logo-300x140.png 300w" sizes="(max-width: 350px) 100vw, 350px" />

## **架构介绍**

&nbsp;

Harbor在架构上主要由6个组件构成：

**Proxy：**Harbor的registry, UI, token等服务，通过一个前置的反向代理统一接收浏览器、Docker客户端的请求，并将请求转发给后端不同的服务。

**Registry：** 负责储存Docker镜像，并处理dockerpush/pull 命令。由于我们要对用户进行访问控制，即不同用户对Docker image有不同的读写权限，Registry会指向一个token服务，强制用户的每次docker pull/push请求都要携带一个合法的token,Registry会通过公钥对token 进行解密验证。

**Core services：** 这是Harbor的核心功能，主要提供以下服务：

<ul class="list-paddingleft-2">
  <li>
    UI：提供图形化界面，帮助用户管理registry上的镜像（image）, 并对用户进行授权。
  </li>
  <li>
    webhook：为了及时获取registry 上image状态变化的情况， 在Registry上配置webhook，把状态变化传递给UI模块。
  </li>
  <li>
    token服务：负责根据用户权限给每个docker push/pull命令签发token.Docker 客户端向Regiøstry服务发起的请求,如果不包含token，会被重定向到这里，获得token后再重新向Registry进行请求。
  </li>
</ul>

**Database：**为coreservices提供数据库服务，负责储存用户权限、审计日志、Docker image分组信息等数据。

**Job Services：**提供镜像远程复制功能，可以把本地镜像同步到其他Harbor实例中。

**Log collector：**为了帮助监控Harbor运行，负责收集其他组件的log，供日后进行分析。

## **技术实现**

Harbor的每个组件都是以Docker 容器的形式构建的，因此很自然地，我们使用Docker Compose来对它进行部署。

<img class="aligncenter size-full wp-image-639" src="http://images.wanglijie.cn/public/img/posts/2016/07/1-harobr架构.png" alt="1-harobr架构" width="640" height="315" srcset="http://images.wanglijie.cn/public/img/posts/2016/07/1-harobr架构.png 640w, http://images.wanglijie.cn/public/img/posts/2016/07/1-harobr架构-300x148.png 300w" sizes="(max-width: 640px) 100vw, 640px" />

在源代码中(<a><strong>https://github.com/vmware/harbor</strong></a>), 用于部署Harbor的Docker Compose 模板位于 /Deployer/docker-compose.yml. 打开这个模板文件，会发现Harbor由6个容器组成：

<ul class="list-paddingleft-2">
  <li>
    <strong>proxy:</strong> 由Nginx 服务器构成的反向代理。
  </li>
  <li>
    <strong>registry:</strong>由Docker官方的开源registry 镜像构成的容器实例。
  </li>
  <li>
    <strong>ui: </strong>即架构中的core services, 构成此容器的代码是Harbor项目的主体。
  </li>
  <li>
    <strong>mysql:</strong> 由官方MySql镜像构成的数据库容器。
  </li>
  <li>
    <strong><strong>job services:</strong></strong> 通过状态机机制实现远程镜像复制功能，包括镜像删除也可以同步到远端Harbor实例。
  </li>
  <li>
    <strong>log:</strong> 运行着rsyslogd的容器，通过log-driver的形式收集其他容器的日志。
  </li>
</ul>

这几个容器通过Docker的DNS形式连接在一起，这样，在容器之间可以通过容器名字互相访问。对终端用户而言，只需要暴露proxy（即Nginx）的服务端口。

## **工作原理**

下面以两个Docker 命令为例，讲解主要组件之间如何协同工作。

**docker login**

假设我们将Harbor部署在IP 为192.168.1.10的虚机上。用户通过docker login命令向这个Harbor服务发起登录请求：

\# docker login 192.168.1.10

当用户输入所需信息并点击回车后，Docker 客户端会向地址“192.168.1.10/v2/” 发出HTTP GET请求。Harbor的各个容器会通过以下步骤处理：

<img class="aligncenter size-full wp-image-641" src="http://images.wanglijie.cn/public/img/posts/2016/07/2-docker-login.png" alt="2-docker-login" width="640" height="211" srcset="http://images.wanglijie.cn/public/img/posts/2016/07/2-docker-login.png 640w, http://images.wanglijie.cn/public/img/posts/2016/07/2-docker-login-300x99.png 300w" sizes="(max-width: 640px) 100vw, 640px" />

**(a)** 首先，这个请求会由监听80端口的proxy容器接收到。根据预先设置的匹配规则，容器中的Nginx会将请求转发给后端的registry 容器；

**(b)** 在registry容器一方，由于配置了基于token的认证，registry会返回错误代码401，提示Docker客户端访问token服务绑定的URL。在Harbor中，这个URL指向Core Services；

**(c)** Docker  客户端在接到这个错误代码后，会向token服务的URL发出请求，并根据HTTP协议的Basic Authentication规范，将用户名密码组合并编码，放在请求头部(header)；

**(d)** 类似地，这个请求通过80端口发到proxy容器后，Nginx会根据规则把请求转发给ui容器，ui容器监听token服务网址的处理程序接收到请求后，会将请求头解码，得到用户名、密码；

**(e)** 在得到用户名、密码后，ui容器中的代码会查询数据库，将用户名、密码与mysql容器中的数据进行比对（注：ui 容器还支持LDAP的认证方式，在那种情况下ui会试图和外部LDAP服务进行通信并校验用户名/密码)。比对成功，ui容器会返回表示成功的状态码，并用密钥生成token，放在响应体中返回给Docker 客户端。

至此，一次docker login 成功地完成了，Docker客户端会把步骤(c)中编码后的用户名密码保存在本地的隐藏文件中。

**docker push的流程**

<img class="aligncenter size-full wp-image-642" src="http://images.wanglijie.cn/public/img/posts/2016/07/3-docker-push的流程-1.png" alt="3-docker-push的流程" width="640" height="262" srcset="http://images.wanglijie.cn/public/img/posts/2016/07/3-docker-push的流程-1.png 640w, http://images.wanglijie.cn/public/img/posts/2016/07/3-docker-push的流程-1-300x123.png 300w" sizes="(max-width: 640px) 100vw, 640px" />

（我们省去proxy转发的步骤，上图描述了docker push的过程中各组件的通信）

用户登录成功后用docker push命令向Harbor 推送一个Docker image：

\# docker push 192.168.1.10/library/hello-world

**(a)** 首先，docker 客户端会重复login的过程，首先发送请求到registry,之后得到token 服务的地址；

**(b)** 之后，Docker 客户端在访问ui容器上的token服务时会提供额外信息，指明它要申请一个对imagelibrary/hello-world进行push操作的token；

**(c)** token 服务在经过Nginx转发得到这个请求后，会访问数据库核实当前用户是否有权限对该image进行push。如果有权限，它会把image的信息以及push动作进行编码，并用私钥签名，生成token返回给Docker客户端；

**(d)** 得到token之后Docker客户端会把token放在请求头部，向registry发出请求，试图开始推送image。Registry 收到请求后会用公钥解码token并进行核对，一切成功后，image的传输就开始了。

本文并未涉及Harbor项目本身的配置、部署，这方面请参考Harbor在github上的文档：**<a>https://github.com/vmware/harbor</a>**

_原文: 姜坦/Henry 运维帮_

转载请注明：[自动化运维](http://www.wanglijie.cn) &raquo; [Harbor 企业级开源容器Registry](http://www.wanglijie.cn/2016/07/harbor-%e4%bc%81%e4%b8%9a%e7%ba%a7%e5%bc%80%e6%ba%90%e5%ae%b9%e5%99%a8registry.html)