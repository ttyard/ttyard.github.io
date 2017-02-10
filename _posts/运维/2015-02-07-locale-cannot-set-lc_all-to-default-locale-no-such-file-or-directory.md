---
id: 270
title: 'locale: Cannot set LC_ALL to default locale: No such file or directory'
date: 2015-02-07T20:20:51+00:00
author: 深海游鱼
layout: post
guid: http://www.aixiuyun.com/?p=270
permalink: /2015/02/locale-cannot-set-lc_all-to-default-locale-no-such-file-or-directory.html
views:
  - "477"
categories:
  - 运维
tags:
  - Linux
---
在Ubuntu安装软件时，系统提示报错信息如下：
  
perl: warning: Setting locale failed.
  
perl: warning: Please check that your locale settings:
	  
LANGUAGE = (unset),
	  
LC_ALL = (unset),
	  
LC\_TIME = &#8220;zh\_CN.UTF-8&#8221;,
	  
LC\_MONETARY = &#8220;zh\_CN.UTF-8&#8221;,
	  
LC\_ADDRESS = &#8220;zh\_CN.UTF-8&#8221;,
	  
LC\_TELEPHONE = &#8220;zh\_CN.UTF-8&#8221;,
	  
LC\_NAME = &#8220;zh\_CN.UTF-8&#8221;,
	  
LC\_MEASUREMENT = &#8220;zh\_CN.UTF-8&#8221;,
	  
LC\_IDENTIFICATION = &#8220;zh\_CN.UTF-8&#8221;,
	  
LC\_NUMERIC = &#8220;zh\_CN.UTF-8&#8221;,
	  
LC\_PAPER = &#8220;zh\_CN.UTF-8&#8221;,
	  
LANG = &#8220;en_US.UTF-8&#8221;
      
are supported and installed on your system.
  
perl: warning: Falling back to the standard locale (&#8220;C&#8221;).
  
locale: Cannot set LC_ALL to default locale: No such file or directory

最佳方法是
  
修改/var/lib/locales/supported.d/local，追加一行如下内容：

<pre class="prettyprint linenums" >zh_CN.UTF-8 UTF-8
</pre>

然后：

<pre class="prettyprint linenums" >sudo locale-gen
sudo dpkg-reconfigure locales
</pre>

就完美解决了。

转载请注明：[自动化运维](http://www.wanglijie.cn) &raquo; [locale: Cannot set LC_ALL to default locale: No such file or directory](http://www.wanglijie.cn/2015/02/locale-cannot-set-lc_all-to-default-locale-no-such-file-or-directory.html)