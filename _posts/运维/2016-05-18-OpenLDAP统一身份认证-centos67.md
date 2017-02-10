---
id: 443
title: 'OpenLDAP统一身份认证 [CentOS6/7]'
date: 2016-05-18T14:09:38+00:00
author: 深海游鱼
layout: post
guid: http://www.wanglijie.cn/?p=443
permalink: '/2016/05/openldap%e7%bb%9f%e4%b8%80%e8%ba%ab%e4%bb%bd%e8%ae%a4%e8%af%81-centos67.html'
views:
  - "76"
categories:
  - 运维
tags:
  - Linux
---
本文CentOS6/CentOS7 Linux系统平台，构建OpenLDAP的统一身份认证和双主从同步架构。
  
即两台LDAP服务器互为主、备，其中任一节点数据更新，将自动同步到另外一个节点上，从而达到数据备份，避免了单点故障。
  
[<img src="http://www.wanglijie.cn/wp-content/uploads/2016/05/1064028.png" alt="1064028" width="471" height="352" class="aligncenter size-full wp-image-451" srcset="http://www.wanglijie.cn/wp-content/uploads/2016/05/1064028.png 471w, http://www.wanglijie.cn/wp-content/uploads/2016/05/1064028-300x224.png 300w" sizes="(max-width: 471px) 100vw, 471px" />](http://www.wanglijie.cn/wp-content/uploads/2016/05/1064028.png)

## 1.OpenLDAP安装

<pre class="prettyprint linenums" >#CentOS7或CentOS7
[root@centos ~]# yum install -y openldap-servers openldap-clients
[root@centos ~]# cp /usr/share/openldap-servers/DB_CONFIG.example /var/lib/ldap/DB_CONFIG
[root@centos ~]# chown ldap. /var/lib/ldap/DB_CONFIG
</pre>

启动OpenLDAP服务，并添加到开机启动列表中

<pre class="prettyprint linenums" >#CentOS6
[root@centos ~]# service slapd start
[root@centos ~]# chkconfig slapd on
</pre>

<pre class="prettyprint linenums" >#CentOS7
[root@centos ~]# systemctl start slapd
[root@centos ~]# systemctl enable slapd
</pre>

## 2.设置LDAP管理员密码（admin）

使用slappasswd 生成加密后的密码

<pre class="prettyprint linenums" >[root@centos ~]# slappasswd -s admin
New password:
Re-enter new password:
{SSHA}nKn/k9v72WiAF28quBZiGwBHyINg8rgF
</pre>

首先查找当前系统ldap的配置数据库名称，注意CentOS7默认采用hdb数据库，CentOS6默认采用bdb数据库

<pre class="prettyprint linenums" >#CentOS6
[root@centos6 ~]# sudo slapcat -b cn=config | grep "^dn: olcDatabase="
dn: olcDatabase={-1}frontend,cn=config
dn: olcDatabase={0}config,cn=config
dn: olcDatabase={1}monitor,cn=config
dn: olcDatabase={2}bdb,cn=config
</pre>

<pre class="prettyprint linenums" >#CentOS7
[root@centos7 ~]# sudo slapcat -b cn=config | grep "^dn: olcDatabase="
dn: olcDatabase={-1}frontend,cn=config
dn: olcDatabase={0}config,cn=config
dn: olcDatabase={1}monitor,cn=config
dn: olcDatabase={2}hdb,cn=config
</pre>

通过ldap api，将密码写入ldap配置数据库，注意将olcDatabase修改成对应系统的配置数据库名称：

<pre class="prettyprint linenums" >#CentOS7
[root@centos6 ~]# ldapmodify -Q -Y EXTERNAL -H ldapi:/// &lt;&lt;EOF
dn: olcDatabase={2}bdb,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}nKn/k9v72WiAF28quBZiGwBHyINg8rgF
EOF
</pre>

<pre class="prettyprint linenums" >#CentOS6
[root@centos7 ~]# ldapmodify -Q -Y EXTERNAL -H ldapi:/// &lt;&lt;EOF
dn: olcDatabase={2}hdb,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}nKn/k9v72WiAF28quBZiGwBHyINg8rgF
EOF
</pre>

### 注意：

  1. (1).两种方式新增的密码，在olcDatabase={2}bdb.ldif或olcDatabase={2}hdb.ldif文件中展现的形式略微不同。通过ldapi修改的密码，在ldif文件中，将不显示加密方式：
  
    [<img src="http://www.wanglijie.cn/wp-content/uploads/2016/05/clipboard.png" alt="password" width="495" height="54" class="aligncenter size-full wp-image-454" srcset="http://www.wanglijie.cn/wp-content/uploads/2016/05/clipboard.png 495w, http://www.wanglijie.cn/wp-content/uploads/2016/05/clipboard-300x33.png 300w" sizes="(max-width: 495px) 100vw, 495px" />](http://www.wanglijie.cn/wp-content/uploads/2016/05/clipboard.png)
  2. (2).请首先修改LDAP域后在修改密码。否则可能会导致创建的密码无法登录的情况。
  3. (3).CentOS6和CentOS7默认数据库存储方式不一样。CentOS7采用hdb，CentOS6采用bdb</li> 

## 3.配置OpenLDAP系统日志

修改slapd日志级别

<pre class="prettyprint linenums" >[root@centos7 ~]#ldapmodify -Y EXTERNAL -H ldapi:/// &lt;&lt;EOF
dn:cn=config
changetype:modify
replace:olcLogLevel
olcLogLevel:stats
EOF
</pre>

通过系统的rsyslog配置日志保存的文件

<pre class="prettyprint linenums" >vi /etc/rsyslog.conf
</pre>

在文件底部加入如下内容，然后重启rsyslog和slapd 服务：

<pre class="prettyprint linenums" >local4.* /var/log/slapd.log
</pre>

### 注意：

你也可以在/etc/rsyslog.d目录下名为openldap.conf，将上面的内容写入该文件后重启效果是一样的。

<pre class="prettyprint linenums" >#CentOS7
[root@centos7 ~]# systemctl restart rsyslog.service
[root@centos7 ~]# systemctl restart slapd
</pre>

<pre class="prettyprint linenums" >#CentOS6
[root@centos6 ~]# service rsyslog restart
[root@centos6 ~]# service slapd restart
</pre>

## 4.创建基础组织树配置ldif文件

<pre class="prettyprint linenums" >cat > /tmp/template.ldif &lt;&lt; EOF
dn: dc=aixiuyun,dc=com
objectclass: dcObject
objectclass: organization
o: aixiuyun com
dc: aixiuyun

dn: ou=People,dc=aixiuyun,dc=com
objectClass: organizationalUnit
objectClass: top
ou: People

dn: ou=Groups,dc=aixiuyun,dc=com
objectClass: organizationalUnit
objectClass: top
ou: Groups

dn: cn=Manager,dc=aixiuyun,dc=com
objectclass: organizationalRole
cn: Manager
EOF
</pre>

### 将LDIF文件应用到LDAP数据库中

<pre class="prettyprint linenums" >[root@centos7 ~]# ldapadd -x -D "cn=Manager,dc=aixiuyun,dc=com" -W -f  /tmp/template.ldif
Enter LDAP Password:
adding new entry "dc=aixiuyun,dc=com"

adding new entry "ou=People,dc=aixiuyun,dc=com"

adding new entry "ou=Groups,dc=aixiuyun,dc=com"

adding new entry "cn=Manager,dc=aixiuyun,dc=com"
</pre>

### 添加成功后，可以通过如下命令进行查询

<pre class="prettyprint linenums" >[root@centos7 ~]# ldapsearch -x -b 'dc=aixiuyun,dc=com' '(objectclass=*)'
# extended LDIF
#
# LDAPv3
# base &lt;dc=aixiuyun,dc=com> with scope subtree
# filter: (objectclass=*)
# requesting: ALL
#

# aixiuyun.com
dn: dc=aixiuyun,dc=com
objectClass: dcObject
objectClass: organization
o: aixiuyun com
dc: aixiuyun

# People, aixiuyun.com
dn: ou=People,dc=aixiuyun,dc=com
objectClass: organizationalUnit
objectClass: top
ou: People
......
# numResponses: 5
# numEntries: 4
</pre>

## 5.LDAP系统安全加固

### 禁用匿名登录

<pre class="prettyprint linenums" >[root@centos7 ~]# ldapmodify -Q -Y EXTERNAL -H ldapi:/// &lt;&lt;EOF
dn: cn=config
changetype: modify
add: olcDisallows
olcDisallows: bind_anon
EOF
 
[root@centos7 ~]# ldapmodify -Q -Y EXTERNAL -H ldapi:/// &lt;&lt;EOF
dn: olcDatabase={-1}frontend,cn=config
changetype: modify
add: olcRequires
olcRequires: authc
EOF
</pre>

允许用户自己修改密码

<pre class="prettyprint linenums" >#CentOS6
[root@centos6 ~]# ldapmodify -Q -Y EXTERNAL -H ldapi:/// &lt;&lt;EOF
dn: olcDatabase{2}bdb,cn=config
changetype: modify
replace: olcAccess              
access to attrs=userPassword
 by self write  
 by anonymous auth  
 by users none
EOF
</pre>

<pre class="prettyprint linenums" >#CentOS7
[root@centos7 ~]# ldapmodify -Q -Y EXTERNAL -H ldapi:/// &lt;&lt;EOF
dn: olcDatabase{2}hdb,cn=config
changetype: modify
replace: olcAccess              
access to attrs=userPassword
 by self write  
 by anonymous auth  
 by users none
EOF
</pre>

注意：by前面有必须要有一个空格，否则会报错。

### 启用slapd TLS

复制CA中级证书，服务器证书到/etc/openldap/certs

<pre class="prettyprint linenums" >[root@centos7 ~]# mkdir /etc/openldap/certs
[root@centos7 ~]# cp /etc/pki/tls/certs/server.key \
/etc/pki/tls/certs/server.crt \
/etc/pki/tls/certs/ca-bundle.crt \
/etc/openldap/certs/ 
[root@centos7 ~]# chown ldap. -R /etc/openldap/certs

cat > /tmp/mod_ssl.ldif &lt;&lt; EOF
# create new
dn: cn=config
changetype: modify
add: olcTLSCACertificateFile
olcTLSCACertificateFile: /etc/openldap/certs/ca-bundle.crt
-
replace: olcTLSCertificateFile
olcTLSCertificateFile: /etc/openldap/certs/server.crt
-
replace: olcTLSCertificateKeyFile
olcTLSCertificateKeyFile: /etc/openldap/certs/server.key
EOF

[root@centos7 ~]# ldapmodify -Y EXTERNAL -H ldapi:/// -f  /tmp/mod_ssl.ldif
</pre>

修改/etc/sysconfig/slapd，加入ldaps:///

<pre class="prettyprint linenums" >[root@centos7 ~]# vi /etc/sysconfig/slapd
# line 9: add
SLAPD_URLS="ldapi:/// ldap:/// ldaps:/// "

[root@centos7 ~]# systemctl restart slapd 
</pre>

### 启用本地Client使用ldaps访问ldap

<pre class="prettyprint linenums" >[root@centos7 ~]#  echo "TLS_REQCERT allow" &lt;&lt; /etc/openldap/ldap.conf 
</pre>

### 配置nslcd服务使用ldaps，此服务用于集成本地账户或应用系统登录

<pre class="prettyprint linenums" >[root@centos7 ~]# echo "tls_reqcert allow" &lt;&lt; /etc/nslcd.conf 
[root@centos7 ~]# authconfig --enableldaptls --update 
</pre>

## 6.OpenLDAP主从复制

### 在LDAP Master节点启用同步模块

<pre class="prettyprint linenums" >[root@centos7-Master ~]# cat >  /temp/mod_syncprov.ldif &lt;&lt; EOF
# create new
dn: cn=module,cn=config
objectClass: olcModuleList
cn: module
olcModulePath: /usr/lib64/openldap
olcModuleLoad: syncprov.la
EOF

[root@centos7-Master ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f /temp/mod_syncprov.ldif

[root@centos7-Master ~]# cat >  /temp/syncprov.ldif &lt;&lt; EOF
# create new
dn: olcOverlay=syncprov,olcDatabase={2}hdb,cn=config
objectClass: olcOverlayConfig
objectClass: olcSyncProvConfig
olcOverlay: syncprov
olcSpSessionLog: 100
EOF

[root@centos7-Master ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f /temp/syncprov.ldif

[root@centos7-slave ~]# cat >  /temp/syncrepl.ldif &lt;&lt; EOF
dn: olcDatabase={2}hdb,cn=config
changetype: modify
add: olcSyncRepl
olcSyncRepl: rid=001
  provider=ldap://10.12.49.44:389/
  bindmethod=simple
  binddn="cn=Manager,dc=aixiuyun,dc=com"
  credentials=password
  searchbase="dc=aixiuyun,dc=com"
  scope=sub
  schemachecking=on
  type=refreshAndPersist
  retry="30 5 300 3"
  interval=00:00:05:00
EOF

[root@centos7-slave ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f /temp/syncrepl.ldif  
</pre>

### 注意：

  1. (1).在olcSyncRepl下面的内容前面要保持有空格。
  2. (2).密码若包含特殊符号不需要使用引号，直接将内容填入credentials

### 设置LDAP客户端使用多个ldap服务器

<pre class="prettyprint linenums" >[root@centos7 ~]# authconfig --ldapserver=ldap1.aixiuyun.com,ldap2.aixiuyun.com --update 
</pre>

### 分别重启openldap服务

<pre class="prettyprint linenums" >[root@centos7 ~]# systemctl restart slapd
</pre>

## 7.OpenLDAP多主同步

多主同步，是有两台以上LDAP服务器，互为主从接口，在其中任意一台修改数据都可以同步到另外一台服务器上。
  
首先按照上文中的内容，分别在两台ldap服务器上加载并启用同步模块。

<pre class="prettyprint linenums" >[root@centos7 ~]# cat >  /temp/mod_syncprov.ldif &lt;&lt; EOF
# create new
dn: cn=module,cn=config
objectClass: olcModuleList
cn: module
olcModulePath: /usr/lib64/openldap
olcModuleLoad: syncprov.la
EOF

[root@centos7 ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f /temp/mod_syncprov.ldif

[root@centos7 ~]# cat >  /temp/syncprov.ldif &lt;&lt; EOF
# create new
dn: olcOverlay=syncprov,olcDatabase={2}hdb,cn=config
objectClass: olcOverlayConfig
objectClass: olcSyncProvConfig
olcOverlay: syncprov
olcSpSessionLog: 100
EOF

[root@centos7 ~]# ldapadd -Y EXTERNAL -H ldapi:/// -f /temp/syncprov.ldif
</pre>

### 在所有的ldap服务上，配置同步的数据源服务器，注意在服务器上配置不同的olcServerID和provider数据源服务器的地址。

<pre class="prettyprint linenums" >[root@centos7 ~]# cat > /temp/master01.ldif  &lt;&lt;EOF
dn: cn=config
changetype: modify
replace: olcServerID
# specify uniq ID number on each server
olcServerID: 0

dn: olcDatabase={2}hdb,cn=config
changetype: modify
add: olcSyncRepl
olcSyncRepl: rid=001
  provider=ldap://10.12.49.44:389/
  bindmethod=simple  
  binddn="cn=Manager,dc=aixiuyun,dc=com"
  credentials=password
  searchbase="dc=aixiuyun,dc=com"
  scope=sub
  schemachecking=on
  type=refreshAndPersist
  retry="30 5 300 3"
  interval=00:00:05:00
-
add: olcMirrorMode
olcMirrorMode: TRUE

dn: olcOverlay=syncprov,olcDatabase={2}hdb,cn=config
changetype: add
objectClass: olcOverlayConfig
objectClass: olcSyncProvConfig
olcOverlay: syncprov
EOF

[root@centos7 ~]# ldapmodify -Y EXTERNAL -H ldapi:/// -f /temp/master01.ldif 
</pre>

### 备注：

  1. (1) olcSyncRepl为数据源服务器ID
  2. (2)provider 指定不同的LDAP服务器URI
  3. (3)binddn 具有读取目录权限的用户
  4. (4)credentials 该用户的密码
  5. (5)scope=sub 包含子树
  6. (6)retry 重试的时间间隔，格式如下
      
    \[retry interval\] \[retry times\] \[interval of re-retry\] \[re-retry times\]
       
    retry=&#8221;30 5 300 3&#8243; 
  7. (7)interval= 同步的间隔时间
       
    interval=00:00:05:00 

### 设置LDAP客户端使用多个ldap服务器

<pre class="prettyprint linenums" >[root@centos7 ~]# authconfig --ldapserver=ldap1.aixiuyun.com,ldap2.aixiuyun.com --update 
</pre>

转载请注明：[自动化运维](http://www.wanglijie.cn) &raquo; [OpenLDAP统一身份认证 [CentOS6/7]](http://www.wanglijie.cn/2016/05/openldap%e7%bb%9f%e4%b8%80%e8%ba%ab%e4%bb%bd%e8%ae%a4%e8%af%81-centos67.html)