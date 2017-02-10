---
id: 373
title: 构建Linux Skype Message消息推送API服务（Zabbix集成告警）
date: 2015-08-14T11:09:36+00:00
author: 深海游鱼
layout: post
guid: http://www.aixiuyun.com/?p=373
permalink: '/2015/08/%e6%9e%84%e5%bb%balinux-skype-message%e6%b6%88%e6%81%af%e6%8e%a8%e9%80%81api%e6%9c%8d%e5%8a%a1.html'
views:
  - "595"
categories:
  - 运维
tags:
  - 监控
---
<h2>简介</h2>
<p>在工作中，由于监控系统需要能够通过即时聊天工具发送告警的信息，同时在工作中主要使用Skype进行内部的沟通交流，因此萌发了将Zabbix与Skype进行整合的想法。<br />
本文运行环境基于Ubuntu 14.04 LTS，其他Linux平台安装方式可能略微不同。可以将Skype API服务器部署在Linux和Windows服务器上，也可以和Zabbix部署在同一台机器上面。只需要能够顺利调用http://IP:5000/msg接口即可。</p>
<h3>系统架构说明:</h3>
<p style="text-align: center;"><a href="http://www.wanglijie.cn/wp-content/uploads/2015/08/1-skype-架构图.png"><img class="aligncenter  wp-image-374" src="http://www.wanglijie.cn/wp-content/uploads/2015/08/1-skype-架构图.png" alt="1-skype-架构图" width="681" height="354" /></a><br />
图1-Skype架构图</p>
<h2>一、 安装基础环境和Skype</h2>
<p>在Linux系统中运行Skype需要使用xvfb启动x11图形界面才能正常运行。所有在ubuntu系统中，使用下面的命令进行安装，具体如下：</p>
<pre class="prettyprint linenums">#更新系统
root@ubuntu14:~# apt-get update -y
#升级系统的软件
root@ubuntu14:~# apt-get upgrade -y
#安装Skype基础环境和依赖包
root@ubuntu14:~# apt-get install -y xvfb fluxbox x11vnc dbus libasound2 libqt4-dbus libqt4-network libqtcore4 libqtgui4 libxss1 libpython2.7 libqt4-xml libaudio2 fontconfig liblcms1 lib32stdc++6 libc6-i386 lib32gcc1 nano python-virtualenv
#安装x11字体
root@ubuntu14:~# apt-get install xfonts-100dpi xfonts-75dpi xfonts-scalable xfonts-cyrillic
#下载Skype安装包，Ubuntu x64
root@ubuntu14:~# wget http://www.skype.com/go/getskype-linux-beta-ubuntu-64 -O skype-linux-beta.deb
#下载完毕后，安装Skype安装包
root@ubuntu14:/home/skype#dpkg -i skype-linux-beta.deb

</pre>
<p>Skype Web下载网址：<br />
http://www.skype.com/en/download-skype/skype-for-computer/<br />
如果安装过程中出现依赖包报错无法安装的情况，可以直接使用下面的命令安装依赖包后继续安装。</p>
<pre class="prettyprint linenums">root@ubuntu14:/home/skype# dpkg -i skype-ubuntu-precise_4.3.0.37-1_i386_ubuntu-64.deb 
Selecting previously unselected package skype.
(Reading database ... 64536 files and directories currently installed.)
Preparing to unpack skype-ubuntu-precise_4.3.0.37-1_i386_ubuntu-64.deb ...
Unpacking skype (4.3.0.37-1) ...
dpkg: dependency problems prevent configuration of skype:
 skype depends on libc6 (&gt;= 2.3.6-6~).
 skype depends on libc6 (&gt;= 2.7).
 skype depends on libgcc1 (&gt;= 1:4.1.1).
 skype depends on libqt4-dbus (&gt;= 4:4.5.3).
 skype depends on libqt4-network (&gt;= 4:4.8.0).
 skype depends on libqt4-xml (&gt;= 4:4.5.3).
 skype depends on libqtcore4 (&gt;= 4:4.7.0~beta1).
 skype depends on libqtgui4 (&gt;= 4:4.8.0).
 skype depends on libqtwebkit4 (&gt;= 2.2~2011week36).
 skype depends on libstdc++6 (&gt;= 4.2.1).
 skype depends on libx11-6.
 skype depends on libxext6.
 skype depends on libxss1.
 skype depends on libxv1.
 skype depends on libssl1.0.0.
 skype depends on libpulse0.
 skype depends on libasound2-plugins.

dpkg: error processing package skype (--install):
 dependency problems - leaving unconfigured
Processing triggers for mime-support (3.54ubuntu1.1) ...
Errors were encountered while processing:
 skype

</pre>
<p>使用下面命令可以修复:</p>
<pre class="prettyprint linenums">dpkg --add-architecture i386
apt-get update
</pre>
<p>如果采用最小安装，还需要安装如下软件：</p>
<pre class="prettyprint linenums">#如果系统未安装unzip
sudo apt-get install -y unzip
apt-get install -y python-gobject-2
apt-get install -y curl git
</pre>
<h2>二、 VNC远程访问配置Skype</h2>
<p>Sevabot运行需要本地运行Skype程序并正常登录后，sevabot才能够正常通过Python的API调用Skype来发送消息。所有需要通过VNC连接到服务端，对Skype进行配置。</p>
<h3>2.1. 新增skype系统运行账户</h3>
<pre class="prettyprint linenums">#生成32位随机数密码
root@ubuntu14:~#openssl rand -base64 32
yCNjsTF18qTSEt4jmaZ+yhQ/iigCRoTz2YJC4FTMhkw=

root@ubuntu14:~# adduser skype
Adding user `skype' ...
Adding new group `skype' (1001) ...
Adding new user `skype' (1001) with group `skype' ...
Creating home directory `/home/skype' ...
Copying files from `/etc/skel' ...
Enter new UNIX password: 
Retype new UNIX password: 
passwd: password updated successfully
Changing the user information for skype
Enter the new value, or press ENTER for the default
	Full Name []: skype
	Room Number []: 
	Work Phone []: 
	Home Phone []: 
	Other []: 
Is the information correct? [Y/n] y
</pre>
<h3>2.2.下载并解压sevabot</h3>
<pre class="prettyprint linenums">skype@ubuntu14:~$wget https://github.com/opensourcehacker/sevabot/archive/master.zip
skype@ubuntu14:~$unzip master.zip
skype@ubuntu14:~$mv sevabot-master sevabot &amp; cd sevabot
skype@ubuntu14:~/sevabot$chmod +x sevabot ./ -R

</pre>
<h3>2.3 启动xvfb、fluxbox和Skype服务</h3>
<pre class="prettyprint linenums">skype@ubuntu14:~$SERVICES="xvfb fluxbox skype" ~/sevabot/scripts/start-server.sh start
</pre>
<h3>2.4 启动 VNC server服务</h3>
<p>第一次启动VNC服务，程序会要求输入Skype用户的VNC Viewer访问的密码，并保存在~/.x11vnc/passwd，如果要重置密码，只需要删除/home/skype/.x11vnc/passwd文件即可。</p>
<pre class="prettyprint linenums">skype@ubuntu14:~$ ~/sevabot/scripts/start-vnc.sh start
Xvfb is running
fluxbox is running
skype is running
OVERALL STATUS: OK
Starting x11vnc
Enter VNC password: 
Verify password:    
Write password to /home/skype/.x11vnc/passwd?  [y]/n y
</pre>
<h3>2.5 VNC远程登录</h3>
<p>在远程的桌面系统，通过VNC使用skype用户远程登录到服务器，默认端口5900，并在Skype中输入帐号密码，并修改默认的配置。</p>
<p style="text-align: center;"><a href="http://www.wanglijie.cn/wp-content/uploads/2015/08/2-VNC-connection.png"><img class="aligncenter size-full wp-image-375" src="http://www.wanglijie.cn/wp-content/uploads/2015/08/2-VNC-connection.png" alt="2-VNC-connection" width="415" height="214" srcset="http://www.wanglijie.cn/wp-content/uploads/2015/08/2-VNC-connection.png 415w, http://www.wanglijie.cn/wp-content/uploads/2015/08/2-VNC-connection-300x155.png 300w" sizes="(max-width: 415px) 100vw, 415px" /></a><br />
图2-1 桌面客户端VNC Viewer远程登录服务器</p>
<h3>2.6 Skype初始化配置</h3>
<p style="text-align: center;"><a href="http://www.wanglijie.cn/wp-content/uploads/2015/08/3-配置Skype选择默认语言.png"><img class="aligncenter  wp-image-376" src="http://www.wanglijie.cn/wp-content/uploads/2015/08/3-配置Skype选择默认语言.png" alt="3-配置Skype选择默认语言" width="654" height="513" /></a></p>
<p style="text-align: center;">图2-2 配置Skype，选择默认语言</p>
<p style="text-align: center;"><a href="http://www.wanglijie.cn/wp-content/uploads/2015/08/4-skype-login.png"><img class="aligncenter  wp-image-377" src="http://www.wanglijie.cn/wp-content/uploads/2015/08/4-skype-login.png" alt="4-skype-login" width="682" height="530" /></a></p>
<p style="text-align: center;">图2-3 配置登录的用户名和密码</p>
<p>注意：并勾选“Sign me when Skype status”</p>
<h3>2.7 配置Skype隐私选项</h3>
<ul>
<li>No chat history</li>
<li>Only people on my list can write me</li>
<li>Only people on my list can call me</li>
</ul>
<p style="text-align: center;"><a href="http://www.wanglijie.cn/wp-content/uploads/2015/08/5-skype-configuration.png"><img class="aligncenter  wp-image-378" src="http://www.wanglijie.cn/wp-content/uploads/2015/08/5-skype-configuration.png" alt="5-skype-configuration" width="704" height="554" /></a></p>
<p style="text-align: center;">图2-4 Skype Privacy</p>
<h2>三、 安装Sevabot</h2>
<p>首先使用Skype用户登录的服务器。sevabot默认需要使用python virtualenv配置环境。</p>
<h3>安装Sevabot</h3>
<pre class="prettyprint linenums">skype@ubuntu14:~$ cd ~/sevabot/
skype@ubuntu14:~/sevabot$ virtualenv venv
New python executable in venv/bin/python
Installing setuptools, pip...done.
skype@ubuntu14:~/sevabot$ . venv/bin/activate
(venv)skype@ubuntu14:~/sevabot$ 
(venv)skype@ubuntu14:~/sevabot$ python setup.py develop

</pre>
<p>通过setup.py安装脚本，将能自动安装所需要的依赖文件。sevabot提供了Web界面用于查看Skype chat ID，发送消息。默认监听localhost:5000，我们需要修改settings.py修改配置文件。用于启动sevabot。</p>
<pre class="prettyprint linenums">#生成一个随机的密码。
root@ubuntu14:~# openssl rand -base64 32
cHBVRom0lrEHMTe0gCYYsMr1iH89N7sXN9XL7kP1tZk=

skype@ubuntu14:~/sevabot$ cp settings.py.example settings.py
skype@ubuntu14:~/sevabot$vim settings.py

</pre>
<p><a href="http://www.wanglijie.cn/wp-content/uploads/2015/08/6-settings.py_.png"><img class="aligncenter size-full wp-image-379" src="http://www.wanglijie.cn/wp-content/uploads/2015/08/6-settings.py_.png" alt="6-settings.py" width="523" height="318" srcset="http://www.wanglijie.cn/wp-content/uploads/2015/08/6-settings.py_.png 523w, http://www.wanglijie.cn/wp-content/uploads/2015/08/6-settings.py_-300x182.png 300w" sizes="(max-width: 523px) 100vw, 523px" /></a></p>
<p style="text-align: center;">图3-1 settings.py配置</p>
<p>参数说明：<br />
SHARED_SECRET:密钥，用户登录Web控制台和远程调用时使用的密码。<br />
ADMINS：登录Web控制台时的用户名。<br />
HTTP_HOST：程序监听的地址。<br />
HTTP_PORT：程序监听的端口号。</p>
<p>修改完成后，启动sevabot服务：</p>
<pre class="prettyprint linenums">skype@ubuntu14:~/sevabot$ SERVICES=sevabot ~/sevabot/scripts/start-server.sh start
Started Sevabot web server process id 
</pre>
<p>服务启动完毕后，立即登录VNC到服务器，在Skype中接受Skype4Py的API授权请求，并勾选“Remember this selection”<br />
<a href="http://www.wanglijie.cn/wp-content/uploads/2015/08/7-allow-skype4py-api.png"><img class="aligncenter  wp-image-380" src="http://www.wanglijie.cn/wp-content/uploads/2015/08/7-allow-skype4py-api.png" alt="7-allow skype4py api" width="726" height="564" /></a></p>
<p style="text-align: center;">图3-2 Skye4Py API请求授权</p>
<p>配置完毕后，停止VNC Server的服务：</p>
<pre class="prettyprint linenums">skype@ubuntu14:~$ ~/sevabot/scripts/start-vnc.sh start
</pre>
<p>启动完毕后，确认服务和端口正常运行，在浏览器中输入IP:5000即可打开控制台。<br />
<a href="http://www.wanglijie.cn/wp-content/uploads/2015/08/8-http-api-services.png"><img class="aligncenter  wp-image-381" src="http://www.wanglijie.cn/wp-content/uploads/2015/08/8-http-api-services.png" alt="8-http-api-services" width="551" height="635" /></a></p>
<p style="text-align: center;">图3-3 Web控制台登录</p>
<p>在Skype中，将需要与之通讯的好友加入到通讯录中，并向他发送一条“Hello”信息后，相关的Chat ID就会在下面的列表中显示。有时会出现同一个用户有2个Chat Id的问题，现在还不知道问什么。<br />
<a href="http://www.wanglijie.cn/wp-content/uploads/2015/08/9-skype-chatid-viewer.png"><img class="aligncenter  wp-image-382" src="http://www.wanglijie.cn/wp-content/uploads/2015/08/9-skype-chatid-viewer.png" alt="9-skype-chatid-viewer" width="527" height="400" /></a></p>
<p style="text-align: center;">图3-4 Chat ID</p>
<h2>四、 Zabbix集成Sevabot</h2>
<h3> 创建Skype告警脚本</h3>
<p>查看Zabbix Server 告警脚本保存目录：</p>
<pre class="prettyprint linenums">root@ip-172-31-3-76:/home/ubuntu# cat /etc/zabbix/zabbix_server.conf |grep "alert"
#	How often Zabbix will try to send unsent alerts (in seconds).
#	Full path to location of custom alert scripts.
# AlertScriptsPath=${datadir}/zabbix/alertscripts
AlertScriptsPath=/usr/lib/zabbix/alertscripts
</pre>
<p>创建Skype告警脚本skype_send_message.sh</p>
<pre class="prettyprint linenums">#!/bin/sh
#
# Example shell script for sending a message into sevabot
#
# Give command line parameters [chat id] and [message].
# The message is md5 signed with a shared secret specified in settings.py
# Then we use curl do to the request to sevabot HTTP interface.
#
#

chat=$1
msg=${3}
secret="zgjEMYY2G7INW8ZPD8A7o/S+LaxX5vssr5kwdXWZn/g="
msgaddress="http://localhost:5000/msg/"

md5=`echo -n "$chat$msg$secret" | md5sum`

#md5sum prints a '-' to the end. Let's get rid of that.
for m in $md5; do
    break
done

curl $msgaddress --data-urlencode chat="$chat" --data-urlencode msg="$msg" --data-urlencode md5="$m"
</pre>
<h3>新增Zabbix Media Type</h3>
<p>依次打开：Administrator&gt;Media types&gt; Create media type：<br />
<a href="http://www.wanglijie.cn/wp-content/uploads/2015/08/10-zabbix-1.png"><img class="aligncenter  wp-image-383" src="http://www.wanglijie.cn/wp-content/uploads/2015/08/10-zabbix-1.png" alt="10-zabbix-1" width="626" height="199" /></a></p>
<h3>新增人员及Skype Chat ID</h3>
<p>进入Zabbix控制台，依次打开：administrator &gt; Users &gt; Users，选择需要新增Skype Chat ID的成员。<br />
<a href="http://www.wanglijie.cn/wp-content/uploads/2015/08/11-zabbix-21.png"><img class="aligncenter size-full wp-image-395" src="http://www.wanglijie.cn/wp-content/uploads/2015/08/11-zabbix-21.png" alt="11-zabbix-2" width="555" height="393" srcset="http://www.wanglijie.cn/wp-content/uploads/2015/08/11-zabbix-21.png 555w, http://www.wanglijie.cn/wp-content/uploads/2015/08/11-zabbix-21-300x212.png 300w" sizes="(max-width: 555px) 100vw, 555px" /></a></p>
<h3>配置告警发送动作</h3>
<p>依次打开：Configuration &gt; Actions &gt; create action，进入Operations新增：<br />
<a href="http://www.wanglijie.cn/wp-content/uploads/2015/08/13-zabbix-3.png"><img class="aligncenter  wp-image-385" src="http://www.wanglijie.cn/wp-content/uploads/2015/08/13-zabbix-3.png" alt="13-zabbix-3" width="692" height="503" /></a></p>
<p>至此，所有Skype和Zabbix的集成就已经配置完成了。需要注意的是微软可能在未来会封锁Skype第三方私有API，这将会导致本文的配置方法可能在未来会失效。</p>
<p>参考文档：<br />
https://sevabot-skype-bot.readthedocs.org/en/latest/ubuntu.html</p>
<p>转载请注明：<a href="http://www.wanglijie.cn">自动化运维</a> &raquo; <a href="http://www.wanglijie.cn/2015/08/%e6%9e%84%e5%bb%balinux-skype-message%e6%b6%88%e6%81%af%e6%8e%a8%e9%80%81api%e6%9c%8d%e5%8a%a1.html">构建Linux Skype Message消息推送API服务（Zabbix集成告警）</a></p>