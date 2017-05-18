---
id: 556
title: ELK 综合日志归档分析系统(2)-Logstash安装配置
date: 2016-06-30T17:18:48+00:00
author: 深海游鱼
layout: post
guid: http://www.wanglijie.cn/?p=556
permalink: '/2016/06/elk-%e7%bb%bc%e5%90%88%e6%97%a5%e5%bf%97%e5%bd%92%e6%a1%a3%e5%88%86%e6%9e%90%e7%b3%bb%e7%bb%9f2-logstash%e5%ae%89%e8%a3%85%e9%85%8d%e7%bd%ae.html'
views:
  - "26"
image: /wp-content/uploads/2016/06/logstash-logo-1-220x150.png
categories:
  - 运维
tags:
  - ELK Stack
---
本文是继上篇[《ELK 综合日志归档分析系统(1)-Elasticsearch安装配置》](http://www.wanglijie.cn/2016/06/elk-%e7%bb%bc%e5%90%88%e6%97%a5%e5%bf%97%e5%bd%92%e6%a1%a3%e5%88%86%e6%9e%90%e7%b3%bb%e7%bb%9f1-elasticsearch%e5%ae%89%e8%a3%85%e9%85%8d%e7%bd%ae.html)之后的第二篇，将详细介绍ELK之Logstash安装配置，关于基础环境配置，请参考第一篇文章。

## 1.Logstash介绍

Logstash 项目诞生于2009,她是一款非常优秀的日志收集处理框架,主要用于收集、过滤、分析服务器的日志。

## 2.logstash安装

### Ubuntu（APT）在线安装

<pre class="prettyprint linenums">$ wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
$ echo "deb https://packages.elastic.co/logstash/2.3/debian stable main" | sudo tee -a /etc/apt/sources.list
$ sudo apt-get update && sudo apt-get install logstash
</pre>

### RedHat/CentOS（YUM）在线安装

<pre class="prettyprint linenums">$ rpm --import https://packages.elastic.co/GPG-KEY-elasticsearch
$ vim /etc/yum.repos.d/logstash-23.repos
[logstash-2.3]
name=Logstash repository for 2.3.x packages
baseurl=https://packages.elastic.co/logstash/2.3/centos
gpgcheck=1
gpgkey=https://packages.elastic.co/GPG-KEY-elasticsearch
enabled=1

$ yum install logstash
</pre>

## 3. logstash-Indexer配置

通过Logstash建立中心日志收集服务,并启动监听TCP端口,用于接收服务器发送过来的日志,并将日志暂存到Redis中.
  
logstash安装完毕后,默认的配置文件目录在/etc/logstash/conf.d，由于前端日志采集的服务器包括了Windows、Linux。我们建议在Linux系统下使用filebeat，Windows系统使用nxlog来作为Shipper。在日志传输的过程中，采用SSL对数据流量进行加密处理。
  
首先创建用于Filebeat接收日志的配置文件，这里需要使用beats插件，并启用SSL传输加密。

<pre class="prettyprint linenums">$ vim 01-filebeat-input.conf 
input {
beats {
    #监听端口
        port =&gt; 5044
        #启用ssl
        ssl =&gt; true
        ssl_certificate_authorities =&gt; ["/etc/logstash/pki/ca.crt"]
        ssl_certificate =&gt; "/etc/logstash/pki/elk.wanglijie.cn.crt"
        ssl_key =&gt; "/etc/logstash/pki/elk.wanglijie.cn.key"
        ssl_verify_mode =&gt; "force_peer"
    }
}
</pre>

**配置Logstash TCP Input，用于收集Nxlog发送过来的日志**

<pre class="prettyprint linenums">$ vim 02-nxlog-input.conf 
input {
    tcp {
        port =&gt; 5002
        codec =&gt; "json"
        ssl_extra_chain_certs =&gt; ["/etc/logstash/pki/ca.crt"]
        ssl_cert =&gt; "/etc/logstash/pki/elk.wanglijie.cn.cn.crt"
        ssl_key =&gt; "/etc/logstash/pki/elk.wanglijie.cn.key"
        ssl_enable =&gt; true
        ssl_verify =&gt; false
    }
}
</pre>

**将采集的日志数据，发送的Redis中进行暂存.**

<pre class="prettyprint linenums">$ vim 99-output.conf
output {
    redis { host =&gt; "127.0.0.1" data_type =&gt; "list" key =&gt; "logstash" password=&gt;"GZcY*****Vm" }
#    stdout { codec =&gt; rubydebug }
}
</pre>

## 4.Logstash Center配置

在Logstash indexer中,仅用来接收日志文件,并提交到Redis中,然后通过Logstash Center从Redis中获取接收的日志,并对日志进行过滤后,存入Elasticsearch集群.
  
Logstash安装请参考上文。这里主要对Center日志处理，分词等配置进行解析。

**从Redis中取出日志**

这里使用了spiped将处理外网的Redis映射到如本地的7480端口.

<pre class="prettyprint linenums">$ vim 01-input-cloud-redis.conf 
input {
    redis {
        host=&gt; "127.0.0.1"
        data_type =&gt; "list"
        key =&gt; "logstash"
        codec =&gt; json
        port =&gt; 7480
        password=&gt;"GZcY*****Vm"
    }
}
</pre>

**使用GeoIP分析IP地理位置**

<pre class="prettyprint linenums">$ vim 02-geoip.conf 
filter{
       geoip {
          source =&gt; "ip"
          target =&gt; "geoip"
          database =&gt; "/usr/share/GeoIP/GeoLiteCity.dat"
          add_field =&gt; [ "[geoip][coordinates]", "%{[geoip][longitude]}" ]
          add_field =&gt; [ "[geoip][coordinates]", "%{[geoip][latitude]}"  ]
        }
}
</pre>

**Linux syslog日志分析**

<pre class="prettyprint linenums">$ vim 10-syslog.conf 
filter {
  if [type] == "syslog" {
    grok {
      match =&gt; { "message" =&gt; "%{SYSLOGTIMESTAMP:syslog_timestamp} %{SYSLOGHOST:syslog_hostname} %{DATA:syslog_program}(?:\[%{POSINT:syslog_pid}\])?: %{GREEDYDATA:syslog_message}" }
      #match =&gt; { "message" =&gt; "%{SYSLOGLINE}" }
      #add_field =&gt; [ "received_at", "%{@timestamp}" ]
      #add_field =&gt; [ "received_from", "%{host}" ]
    }
    syslog_pri { }
    date {
      match =&gt; [ "syslog_timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
    }
  }

  if [type] == "syslog_cron" {
    grok {
      match =&gt; { "message" =&gt; "%{CRONLOG}" }
    }
    syslog_pri { }
    date {
      match =&gt; [ "syslog_timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
    }
  }  

  if [type] == "syslog_pamsession" {
    grok {
      match =&gt; { "message" =&gt; "%{SYSLOGPAMSESSION}" }
    }
    date {
      match =&gt; [ "syslog_timestamp", "MMM  d HH:mm:ss", "MMM dd HH:mm:ss" ]
    }
  }  
  
}
</pre>

**Apache日志文件处理**

<pre class="prettyprint linenums">$ vim 11-apache-log.conf 
filter {
	 if [type] == "apache_access" {
		grok {
		  match =&gt; { "message" =&gt; "%{COMBINEDAPACHELOG}" }
		}
		geoip {
		  source =&gt; "clientip"
		  target =&gt; "geoip"
		  database =&gt; "/usr/share/GeoIP/GeoLiteCity.dat"
		  add_field =&gt; [ "[geoip][coordinates]", "%{[geoip][longitude]}" ]
		  add_field =&gt; [ "[geoip][coordinates]", "%{[geoip][latitude]}"  ]
		}
		mutate {
		  convert =&gt; [ "[geoip][coordinates]", "float"]
		}
	}
	
	if [type] == "apache_error" {
		grok {
			patterns_dir =&gt; ["/etc/logstash/patterns.d/"]
			#match =&gt; { "message" =&gt; "%{APACHEERRORLOG}" }			
                        match =&gt; { "message" =&gt; "%{HTTPD_ERRORLOG}"}
			overwrite =&gt; ["message"]
		}
	}
}
</pre>

**Tomcat日志处理**

<pre class="prettyprint linenums">$ vim 12-tomcat-log.conf 
filter {
	if [type] == "tomcat_catalina" and [message] !~ /(.+)/ {
		drop { }
	}
	if [type] == "tomcat_catalina" and "multiline" in [tags] {
		grok {
			match =&gt; [ "message", "%{JAVASTACKTRACEPART}" ]
		}
	}
	
	if [type] == "tomcat_access" {
		grok {
			match =&gt; { "message" =&gt; "%{COMMONAPACHELOG}" }
		}
		#Use GeoIP Locate the IP geographical location
		geoip {
		  source =&gt; "clientip"
		  target =&gt; "geoip"
		  database =&gt; "/usr/share/GeoIP/GeoLiteCity.dat"
		  add_field =&gt; [ "[geoip][coordinates]", "%{[geoip][longitude]}" ]
		  add_field =&gt; [ "[geoip][coordinates]", "%{[geoip][latitude]}"  ]
		}
		mutate {
		  convert =&gt; [ "[geoip][coordinates]", "float"]
		}
		date {
			match =&gt; [ "timestamp" , "dd/MMM/yyyy:HH:mm:ss Z" ]
		}
	}
}
</pre>

**Windows 日志分析与处理**

<pre class="prettyprint linenums">$ vim 20-windows-event-log-filter.conf
filter {
 
    if [type] == "WindowsEventLog" {
        mutate {
            lowercase =&gt; [ "EventType", "FileName", "Hostname", "Severity" ]
        }
        mutate {
            rename =&gt; [ "Hostname", "source_host" ]
        }
        mutate {
            gsub =&gt; ["source_host","\.example\.com",""]
        }
        date {
            match =&gt; [ "EventTime", "YYYY-MM-dd HH:mm:ss +0800" ]
	    timezone =&gt; "UTC"
        }
        mutate {
            rename =&gt; [ "Severity", "eventlog_severity" ]
            rename =&gt; [ "SeverityValue", "eventlog_severity_code" ]
            rename =&gt; [ "Channel", "eventlog_channel" ]
            rename =&gt; [ "SourceName", "eventlog_program" ]
            rename =&gt; [ "SourceModuleName", "nxlog_input" ]
            rename =&gt; [ "Category", "eventlog_category" ]
            rename =&gt; [ "EventID", "eventlog_id" ]
            rename =&gt; [ "RecordNumber", "eventlog_record_number" ]
            rename =&gt; [ "ProcessID", "eventlog_pid" ]
            
        }
 
        if [SubjectUserName] =~ "." {
            mutate {
                replace =&gt; [ "AccountName", "%{SubjectUserName}" ]
            }
        }
        if [TargetUserName] =~ "." {
            mutate {
                replace =&gt; [ "AccountName", "%{TargetUserName}" ]
            }
        }
        if [FileName] =~ "." {
            mutate {
                replace =&gt; [ "eventlog_channel", "%{FileName}" ]
            }
        }
 
        mutate {
            lowercase =&gt; [ "AccountName", "eventlog_channel" ]
        }
 
        mutate {
            remove_field =&gt; [ "SourceModuleType", "EventTimeWritten", "EventReceivedTime", "EventType" ]
        }
    }
}
</pre>

**IIS日志访问日志处理**

这里需要配置Nxlog自定义日志收集时的字段。

<pre class="prettyprint linenums">$ vim 21-filter-iis.conf 
filter {
    if [SourceName] == "IIS" {

        if [message] =~ "^#" {
            drop {}
        }

        useragent {
            add_tag =&gt; [ "UA" ]
            source =&gt; "csUser-Agent"
        }
        if "UA" in [tags] {
            mutate {
                rename =&gt; [ "name", "browser_name" ]
            }
        }
        mutate {
            rename =&gt; [ "s-ip","serverip"]            
            rename =&gt; [ "cs-method","method" ]
            rename =&gt; [ "cs-uri-stem","request"]
            rename =&gt; [ "cs-uri-query","uri_query" ]
            rename =&gt; [ "s-port","port"]
            rename =&gt; [ "cs-username","username" ]
            rename =&gt; [ "c-ip","clientip"]
            rename =&gt; [ "cs-Referer","referer"]
            rename =&gt; [ "sc-status","response" ]
            rename =&gt; [ "sc-substatus","substatus"]
            rename =&gt; [ "sc-win32-status","win32-status"]
            rename =&gt; [ "timetaken","time_request" ]
            
        }        
        geoip {
          source =&gt; "clientip"
          target =&gt; "geoip"
          database =&gt; "/usr/share/GeoIP/GeoLiteCity.dat"
          add_field =&gt; [ "[geoip][coordinates]", "%{[geoip][longitude]}" ]
          add_field =&gt; [ "[geoip][coordinates]", "%{[geoip][latitude]}"  ]
        }
        
        mutate {
            remove_field =&gt; [
                "SourceModuleType",
                "cs-Referer",
                "cs-uri-query",
                "cs-username",
                "csUser-Agent",
                "EventReceivedTime"
            ]
        }
    }
}
</pre>

**将日志输出并存入ElasticSearch集群**

<pre class="prettyprint linenums">$ vim 30-lumberjack-output.conf 
output {
	elasticsearch {		
		hosts =&gt; ["10.112.49.38:9200","10.112.49.99:9200","10.112.49.169:9200"]
		codec =&gt; "json"
		#sniffing =&gt; true

	}
	#stdout {
	#	codec =&gt; rubydebug
	#}
}
</pre>

配置完毕后,重启Logstash，并检查日志文件是否有错误。

转载请注明：[自动化运维](http://www.wanglijie.cn) &raquo; [ELK 综合日志归档分析系统(2)-Logstash安装配置](http://www.wanglijie.cn/2016/06/elk-%e7%bb%bc%e5%90%88%e6%97%a5%e5%bf%97%e5%bd%92%e6%a1%a3%e5%88%86%e6%9e%90%e7%b3%bb%e7%bb%9f2-logstash%e5%ae%89%e8%a3%85%e9%85%8d%e7%bd%ae.html)