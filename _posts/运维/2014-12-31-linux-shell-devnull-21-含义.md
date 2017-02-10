---
id: 258
title: 'Linux Shell /dev/null 2>&#038;1 含义'
date: 2014-12-31T13:34:01+00:00
author: 深海游鱼
layout: post
guid: http://www.wanglijie.cn/?p=258
permalink: '/2014/12/linux-shell-devnull-21-%e5%90%ab%e4%b9%89.html'
views:
  - "547"
categories:
  - 运维
tags:
  - Linux  
---
执行命令是

<pre class="prettyprint linenums">/home/admin/demo.sh &gt;/dev/null 2&gt;&1 &
</pre>

对于& 1 更准确的说应该是文件描述符 1,而1标识标准输出，stdout。
  
对于2 ，表示标准错误，stderr。
  
2>&1 的意思就是将标准错误重定向到标准输出。这里标准输出已经重定向到了 /dev/null。那么标准错误也会输出到/dev/null
  
最后一个& 是让程序在后台执行。

为何2>&1要写在后面？

<pre class="prettyprint linenums">command &gt; file 2&gt;&1
</pre>

首先是command > file将标准输出重定向到file中， 2>&1 是标准错误拷贝了标准输出的行为，也就是同样被重定向到file中，最终结果就是标准输出和错误都被重定向到file中。

<pre class="prettyprint linenums">command 2&gt;&1 &gt;file
</pre>

2>&1 标准错误拷贝了标准输出的行为，但此时标准输出还是在终端。>file 后输出才被重定向到file，但标准错误仍然保持在终端。

用strace可以看到：
  
 **1. command > file 2>&1**
  
这个命令中实现重定向的关键系统调用序列是：

<pre class="prettyprint linenums">open(file) == 3
   dup2(3,1)
   dup2(1,2)
   
</pre>

**2. command 2>&1 >file**
  
这个命令中实现重定向的关键系统调用序列是：

<pre class="prettyprint linenums">dup2(1,2)
   open(file) == 3
   dup2(3,1)
</pre>

转载请注明：[自动化运维](http://www.wanglijie.cn) &raquo; [Linux Shell /dev/null 2>&#038;1 含义](http://www.wanglijie.cn/2014/12/linux-shell-devnull-21-%e5%90%ab%e4%b9%89.html)