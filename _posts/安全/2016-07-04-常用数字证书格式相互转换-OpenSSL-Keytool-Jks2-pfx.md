---
id: 584
title: 常用数字证书格式相互转换-OpenSSL/Keytool/Jks2pfx
date: 2016-07-04T22:59:11+00:00
author: 深海游鱼
layout: post
guid: http://www.wanglijie.cn/?p=584
permalink: '/2016/07/%e5%b8%b8%e7%94%a8%e6%95%b0%e5%ad%97%e8%af%81%e4%b9%a6%e6%a0%bc%e5%bc%8f%e7%9b%b8%e4%ba%92%e8%bd%ac%e6%8d%a2-opensslkeytooljks2pfx.html'
views:
  - "23"
image: /wp-content/uploads/2016/07/info05031501-220x150.gif
categories:
  - 运维
tags:
  - 安全
  - OpenSSL
---
本文将主要介绍使用openssl、JKS2PFX、Keytool命令完成常用各类证书格式（.PFX , .cer或crt , .key , .jks ）的相互转换。
  
<img src="http://www.wanglijie.cn/wp-content/uploads/2016/07/info05031501.gif" alt="info05031501" width="233" height="236" class="aligncenter size-full wp-image-590" />

## 1: pfx转换cer，crt，key

导出私钥必须输入密码

<pre class="prettyprint linenums" >openssl pkcs12 -in test.wosign.pfx -nodes -out test.pem</pre>

去除pem证书密码，转换成rsa格式，如果使用DSA格式，只需将rsa替换成dsa

<pre class="prettyprint linenums" >openssl rsa -in test.pem -out test.key
openssl x509 -in test.pem -out test.crt（cer）
</pre>

## 2: key和cer转换为pfx

<pre class="prettyprint linenums" >openssl pkcs12 -export -in client1.crt -inkey client1.key -out client1.pfx
</pre>

### 3: .jks转为.pfx

<pre class="prettyprint linenums" >JKS2PFX server.jks 123456 tomcat exportfile c:\progra~1\Java\jre1.5.0_06\bin (转换后文件存放在相应目录下。)
</pre>

## 4: .pfx转换.jks

<pre class="prettyprint linenums" >Keytool -importkeystore -srckeystore aaa.pfx -destkeystore 173.jks -srcstoretype PKCS12 -deststoretype JKS
</pre>

## 5: 创建PFX格式证书（pkcs12 ）

<pre class="prettyprint linenums" >openssl pkcs12 -export -in my.cer -inkey my.key -out mycert.pfx
</pre>

## 6: 证书编码格式转换

<pre class="prettyprint linenums" >#PEM to DER
openssl x509 -in cert.crt -outform der -out cert.der
#DER to PEM
openssl x509 -in cert.crt -inform der -outform pem -out cert.pem
</pre>

转载请注明：[自动化运维](http://www.wanglijie.cn) &raquo; [常用数字证书格式相互转换-OpenSSL/Keytool/Jks2pfx](http://www.wanglijie.cn/2016/07/%e5%b8%b8%e7%94%a8%e6%95%b0%e5%ad%97%e8%af%81%e4%b9%a6%e6%a0%bc%e5%bc%8f%e7%9b%b8%e4%ba%92%e8%bd%ac%e6%8d%a2-opensslkeytooljks2pfx.html)