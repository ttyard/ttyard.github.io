---
id: 542
title: Redmine 3.2 安装配置指南 CentOS6
date: 2016-06-29T16:21:20+00:00
author: 深海游鱼
layout: post
guid: http://www.wanglijie.cn/?p=542
permalink: '/2016/06/redmine-3-2-%e5%ae%89%e8%a3%85%e9%85%8d%e7%bd%ae%e6%8c%87%e5%8d%97-centos6.html'
views:
  - "31"
image: /wp-content/uploads/2016/06/redmine-logo-220x150.png
categories:
  - 运维
tags:
---
## 1.Redmine简介

Redmine是用Ruby开发的基于WEB的项目管理软件，是用ROR框架开发的一套跨平台项目管理系统，支持多种数据库，有不少自己独特的功能，例如提供WIKI、新闻台等，还可以集成其他版本管理系统和BUG跟踪系统，例如PERFORCE、SVN、CVS、TD等等。这种 WEB 形式的项目管理系统通过“项目（PROJECT）”的形式把成员、任务（问题）、文档、讨论以及各种形式的资源组织在一起，大家参与更新任务、文档等内容来推动项目的进度。

## 2.Ruby基础环境配置（RVM/Ruby/Rails）

### 2.1.安装RVM

<pre class="prettyprint linenums">$ curl -L https://get.rvm.io | sudo bash -s stable
#将用户加入rvm组
#若部分用户无法执行rvm命令，可以通过重启，使其生效
$ usermod -a -G rvm wlj
</pre>

确认rvm版本

<pre class="prettyprint linenums">$ rvm -v
rvm 1.27.0 (latest) by Wayne E. Seguin &lt;wayneeseguin@gmail.com&gt;, Michal Papis &lt;mpapis@gmail.com&gt; [https://rvm.io/]
</pre>

### 2.2.安装Ruby必要的组件

<pre class="prettyprint linenums">$ rvm requirements
</pre>

### 2.3安装Ruby 2.3

Ruby安装目录为/usr/loca/rvm/rubies

<pre class="prettyprint linenums">$ rvm install 2.3
</pre>

### 2.4设置当前Ruby使用版本

<pre class="prettyprint linenums">$ rvm use 2.3 --default
Using /usr/local/rvm/gems/ruby-2.3.0
$ ruby -v
ruby 2.3.0p0 (2015-12-25 revision 53290) [x86_64-linux]
$ rvm -v
rvm 1.27.0 (latest) by Wayne E. Seguin &lt;wayneeseguin@gmail.com&gt;, Michal Papis &lt;mpapis@gmail.com&gt; [https://rvm.io/]
$ gem -v
2.5.1
</pre>

使用Taobao RubyGems镜像,提高安装的速度和成功率,否则可能会因为网络问题无法正常安装软件包

<pre class="prettyprint linenums">$ gem sources --add https://ruby.taobao.org/ --remove https://rubygems.org/
$ gem sources -l
*** CURRENT SOURCES ***

https://ruby.taobao.org
# 请确保只有 ruby.taobao.org
$ bundle config mirror.https://rubygems.org https://ruby.taobao.org
</pre>

### 2.5安装Rails

在生产环境中,可以不安装说明文档，默认Rails安装在/usr/local/rvm/gems/<span class="pln">ruby</span><span class="pun">&#8211;</span><span class="lit">2.3</span><span class="pun">.</span><span class="lit"></span>/gems/目录下。

<pre class="prettyprint linenums">$ gem install rails --no-rdoc --no-ri
</pre>

## 3.安装配置Apache、MySQL、Passenger

### 3.1安装Apache/MySQL

<pre class="prettyprint linenums">yum install httpd mysql mysql-server ruby-mysql mysql-libs mysql-devel
</pre>

### 3.2使用 Gem 安裝 Passenger

<pre class="prettyprint linenums">$ gem install passenger
</pre>

### 3.3安裝 Apache Passenger 模块

<pre class="prettyprint linenums">$ passenger-install-apache2-module
</pre>

如果在安装过程中出现错误，安装程序的提示解决方案处理即可。

### 3.4在Apache中配置Passenger

程序安装过程中会提示将下列配置文件加入Apache 配置文件中。

<pre class="prettyprint linenums">#CentOS 6
vim /etc/httpd/conf.d/passenger.conf
&lt;IfModule mod_passenger.c&gt;
	PassengerRoot /usr/local/rvm/gems/ruby-2.3.0/gems/passenger-5.0.29
	PassengerDefaultRuby /usr/local/rvm/gems/ruby-2.3.0/wrappers/ruby
&lt;/IfModule&gt;
</pre>

### 3.5在Apache中配置Redmine虚拟机

<pre class="prettyprint linenums">#CentOS6
vim /etc/httpd/conf.d/redmine.conf
#强制用户使用https访问站点
&lt;VirtualHost *:80&gt;
  ServerName redmine.wanglijie.cn
  DocumentRoot /opt/redmine/redmine/public
  Redirect "/" "https://redmine.wanglijie.cn/"
&lt;/VirtualHost&gt;

&lt;VirtualHost *:443&gt;
        DocumentRoot /opt/redmine/redmine/public
        ServerName redmine.wanglijie.cn:443

        ErrorLog logs/ssl_redmine_error_log
        TransferLog logs/ssl_redmine_access_log
        LogLevel warn

        SSLEngine on
        SSLProtocol all -SSLv2
        SSLCipherSuite ALL:!ADH:!EXPORT:!SSLv2:RC4+RSA:+HIGH:+MEDIUM:+LOW
        SSLCertificateFile /etc/pki/ssl/redmine.wanglijie.cn/Apache/2_www.crt
        SSLCertificateKeyFile /etc/pki/ssl/redmine.wanglijie.cn/Apache/3_www.key
        SSLCACertificateFile /etc/httpd/conf/ssl.crt/root_bundle.crt

        &lt;Files ~ "\.(cgi|shtml|phtml|php3?)$"&gt;
                SSLOptions +StdEnvVars
        &lt;/Files&gt;

        &lt;Directory /opt/redmine/redmine/public/&gt;
             # This relaxes Apache security settings.
             AllowOverride all
             # MultiViews must be turned off.
             Options -MultiViews
             # Uncomment this if you're on Apache &gt;= 2.4:
             #Require all granted
        &lt;/Directory&gt;
&lt;/VirtualHost&gt;        
</pre>

### 3.6重新载入Apache2配置文件

<pre class="prettyprint linenums">service httpd reload
</pre>

### 3.7 为Redmine创建独立的MySQL用户和数据库

<pre class="prettyprint linenums">$ mysql -u root -p
Enter password:

mysql&gt;CREATE DATABASE redmine CHARACTER SET utf8;
mysql&gt;CREATE USER 'redmine'@'localhost' IDENTIFIED BY 'qBuJEx7xxxsJweX';
mysql&gt;GRANT ALL PRIVILEGES ON redmine.* TO 'redmine'@'localhost';
mysql&gt;flush privileges;
mysql&gt;exit;
</pre>

## 4.安装Redmine

下载Redmine安装包并解压到/opt/redmine/

<pre class="prettyprint linenums">$ wget http://www.redmine.org/releases/redmine-3.3.0.tar.gz
$ tar -xzvf -C /opt/redmine/ redmine-3.3.0.tar.gz
$ ln -s /opt/redmine/redmine-3.3.0 /opt/redmine/redmine
</pre>

修改配置文件,并保存在conf目录

<pre class="prettyprint linenums">$ cp /opt/redmine/redmine/config/database.yml.example /opt/redmine/redmine/config/database.yml
$ vim /opt/redmine/redmine/config/database.yml
production:
  adapter: mysql2
  database: redmine
  host: localhost
  username: redmine
  password: "qBuJEx7xxxsJweX"
  encoding: utf8

$ cp /opt/redmine/redmine/config/configuration.yml.example /opt/redmine/redmine/config/configuration.yml
$ vim /opt/redmine/redmine/config/configuration.yml
default:
  email_delivery:
  attachments_storage_path: /opt/redmine/redmine/attachments_files_redmine
  autologin_cookie_name:
  autologin_cookie_path:
  .....
  scm_darcs_path_regexp:
  scm_filesystem_path_regexp:
  scm_stderr_log_file: /var/log/redmine/redmine_scm_stderr.log
  database_cipher_key:
  rmagick_font_path:
production:
  email_delivery:
    delivery_method: :smtp
    smtp_settings:
      address: 127.0.0.1
      port: 25
      domain: redmine.wanglijie.cn
development:
</pre>

### 创建Redmine附件文件保存目录和日志文件目录

<pre class="prettyprint linenums">$mkdir /opt/redmine/redmine/attachments_files_redmine
$mkdir /var/log/redmine</pre>

### 安装配置Bundler

<pre class="prettyprint linenums">$ gem install bundler --no-rdoc --no-ri
$ yum install zlib* openssl* ImageMagick-devel –y
$ gem install mysql2
#安装rbpdf-font
$ gem install rbpdf-font
$ bundle install --without development test postgresql sqlite</pre>

### 为Rails生成cookies秘钥

<pre class="prettyprint linenums">$ rake generate_secret_token
</pre>

如果发生mysql错误，需要再安装gem的mysql2.在当前安装版本存在如下错误：

解决方法是编辑 expanded.rb,注视465行：

<pre class="prettyprint linenums">vim /usr/local/rvm/gems/ruby-2.3.0/gems/htmlentities-4.3.1/lib/htmlentities/mappings/expanded.rb
 465     #'inodot'         =&gt; 0x0131,   # ı   dup            LATIN SMALL LETTER DOTLESS I
 466     'inodot'         =&gt; 0x0131,   # ı   dup            LATIN SMALL LETTER DOTLESS I
</pre>

### 生成Redmine数据库结构

<pre class="prettyprint linenums">$ cd /opt/redmine/redmine/
$ RAILS_ENV=production rake db:migrate
</pre>

### 生成缺省数据

<pre class="prettyprint linenums">$ RAILS_ENV=production REDMINE_LANG=zh rake redmine:load_default_data
</pre>

重启Redmine 服务，可以直接重启Apache 服务即可全部重启

转载请注明：[自动化运维](http://www.wanglijie.cn) &raquo; [Redmine 3.2 安装配置指南 CentOS6](http://www.wanglijie.cn/2016/06/redmine-3-2-%e5%ae%89%e8%a3%85%e9%85%8d%e7%bd%ae%e6%8c%87%e5%8d%97-centos6.html)