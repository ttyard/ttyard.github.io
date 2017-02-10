---
id: 301
title: 使用Powershell自动过滤暴力破解IP
date: 2015-06-18T14:37:13+00:00
author: 深海游鱼
layout: post
guid: http://www.wanglijie.cn/?p=301
permalink: '/2015/06/%e4%bd%bf%e7%94%a8powershell%e8%87%aa%e5%8a%a8%e8%bf%87%e6%bb%a4%e6%9a%b4%e5%8a%9b%e7%a0%b4%e8%a7%a3ip.html'
views:
  - "354"
categories:
  - 安全
tags:
  - Windows
  - 运维
  - 安全
---
在Windows服务器放置公网后，一直存在来之海外和国内IP针对远程桌面和SQLServer数据库服务帐号密码的暴力破解。于是编写了一个Powershell脚本，通过分析系统安全日志，自动探测攻击来源的IP并加入到Windows防火墙中，从而达到了自动过滤暴力攻击的IP地址。

## 1.开启Windows防火墙

在开始菜单运行中，输入：fw.msc，打开防火墙管理程序，在所有区域启用防火墙规则。

[<img class="aligncenter size-full wp-image-302" src="http://images.wanglijie.cn/public/img/posts/2015/06/图1.jpg" alt="图1" width="920" height="616" srcset="http://images.wanglijie.cn/public/img/posts/2015/06/图1.jpg 920w, http://images.wanglijie.cn/public/img/posts/2015/06/图1-300x201.jpg 300w" sizes="(max-width: 920px) 100vw, 920px" />](http://images.wanglijie.cn/public/img/posts/2015/06/图1.jpg)

## 2.创建防火墙入站规则

依次选择：入站规则&#8211;>新建规则。
  
名称必须为：MY BLACKLIST
  
操作：阻止
  
作用域：远程IP地址，选择“下列IP地址”，初始配置时必须填写一个IP。否则防火墙将阻止所有入站通讯。

[<img class="aligncenter size-full wp-image-303" src="http://images.wanglijie.cn/public/img/posts/2015/06/图2.jpg" alt="图2" width="931" height="618" srcset="http://images.wanglijie.cn/public/img/posts/2015/06/图2.jpg 931w, http://images.wanglijie.cn/public/img/posts/2015/06/图2-300x199.jpg 300w" sizes="(max-width: 931px) 100vw, 931px" />](http://images.wanglijie.cn/public/img/posts/2015/06/图2.jpg)

## 3.下载并运行脚本

将脚本中的10.1.1.20地址修改成远程访问的IP地址，防止将受信任的IP地址防火墙中，导致无法访问。完成在powershell中运行下载的脚本程序。
  
因系统默认只允许运行经过受信任数字签名后的代码，脚本可能无法正常运行，有两个解决方法：
  
1.调整powershell运行安全策略，信任所有本地powershell脚本。

<pre class="prettyprint linenums" >#从网络上下载的脚本执行会提示需要签名
set-executionpolicy remotesigned
#如要恢复强制签名，使用下面的命令
# 验证所有的脚本的签名信息，验证不通过，拒绝执行。
set-executionpolicy AllSigned
</pre>

2.创建自签名证书，对脚本进行签名。
  
参考本站点另外一篇文章：<a href="http://www.wanglijie.cn/2015/06/%E8%87%AA%E7%AD%BE%E5%90%8D%E8%AF%81%E4%B9%A6%E5%AF%B9powershell%E4%BB%A3%E7%A0%81%E7%AD%BE%E5%90%8D/" target="_blank">《自签名证书对Powershell代码签名》</a>

脚本下载地址：<a href="https://github.com/ttyard/powershell-firewall/archive/master.zip" target="_blank">点击此处</a>

3.具体代码如下

<pre class="prettyprint linenums" >#modification your application port number
#Run Secript: powershell.exe -file 
#Cancel Run: CTRL+C
#This code has been used pactera eds data certificate to sign
#If change security policy run all local code: set-executionpolicy remotesigned
#Change 10.1.1.20 to your trust remote access IP
$tick = 0; 
"Start to run at: " + (get-date); 

#fiter 
$regex2 = [regex] "Source Network Address:\t(\d+\.\d+\.\d+\.\d+)"; 
$regex3 = [regex] "CLIENT: (\d+\.\d+\.\d+\.\d+)";
  
while($True) {
	"Running... (tick:" + $tick + ")"; $tick+=1; 
	$blacklist = @();

	#Get System FW Blocked IPs
	$fwDefault=New-object -comObject HNetCfg.FwPolicy2;
	$myruleBlockIPs = ($fwDefault.Rules | where {$_.Name -eq "MY BLACKLIST"} | select -First 1).RemoteAddresses;

	#Port 3389 
	$a = netstat -ant | Select-String ":3389";

	if ($a.count -gt 0) {    
		$ips = get-eventlog Security -Newest 1000 | Where-Object {$_.EventID -eq 4625 -and $_.Message -match "Logon Type:\s+10"} | foreach {
			$m = $regex2.Match($_.Message); $ip = $m.Groups[1].Value; $ip; 
		} | Sort-Object | Tee-Object -Variable list | Get-Unique

		foreach ($ip in $ips) {
			if ((-not ($myruleBlockIPs -match $ip))) {
				$attack_count = ($list | Select-String $ip -SimpleMatch | Measure-Object).count;
				"Found attacking IP on 3389: " + $ip + ", with count: " + $attack_count;
				if ($attack_count -ge 8) {$blacklist = $blacklist + $ip;}
			}
		}
	}

	#Get MSSQLSERVER Audits Failed List
	$mssqlserver=(netstat -ant | Select-String ":1433");

	if ($mssqlserver.count -gt 0) {
		$ips = get-eventlog Application -Newest 1000 | Where-Object {$_.EventID -eq 18456} | foreach {
				$m = $regex3.Match($_.Message);
				$ip = $m.Groups[1].Value;
				$ip;
			} | Sort-Object | Tee-Object -Variable list | Get-Unique

		foreach ($ip in $ips) {
			if ((-not ($blacklist -contains $ip)) -and (-not ($myruleBlockIPs -match $ip))) {
				$attack_count = ($list | Select-String $ip -SimpleMatch | Measure-Object).count;
				"Found attacking MS-SQLServer IP on 1433: " + $ip + ", with count: " + $attack_count;
				if ($attack_count -ge 8) {$blacklist = $blacklist + $ip;}
			}
		}
	}

	#Firewall change 
	foreach ($ip in $blacklist) {
		$fw=New-object -comObject HNetCfg.FwPolicy2;
		$myrule = $fw.Rules | where {$_.Name -eq "MY BLACKLIST"} | select -First 1;   
		if (-not ($myrule.RemoteAddresses -match $ip) -and -not ($ip -like "10.1.1.20")) {
			(get-date)+"   "+"Adding this IP into firewall blocklist: " + $ip;   
			$myrule.RemoteAddresses+=(","+$ip); 
		}
	}

	Wait-Event -Timeout 30 #pause 30 secs
} # end of top while loop.
</pre>

转载请注明：[自动化运维](http://www.wanglijie.cn) &raquo; [使用Powershell自动过滤暴力破解IP](http://www.wanglijie.cn/2015/06/%e4%bd%bf%e7%94%a8powershell%e8%87%aa%e5%8a%a8%e8%bf%87%e6%bb%a4%e6%9a%b4%e5%8a%9b%e7%a0%b4%e8%a7%a3ip.html)