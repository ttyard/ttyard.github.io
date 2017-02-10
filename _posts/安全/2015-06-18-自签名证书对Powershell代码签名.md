---
id: 295
title: 自签名证书对Powershell代码签名
date: 2015-06-18T10:40:08+00:00
author: 深海游鱼
layout: post
guid: http://www.aixiuyun.com/?p=295
permalink: '/2015/06/%e8%87%aa%e7%ad%be%e5%90%8d%e8%af%81%e4%b9%a6%e5%af%b9powershell%e4%bb%a3%e7%a0%81%e7%ad%be%e5%90%8d.html'
views:
  - "397"
categories:
  - 安全
tags:
  - Windows
  - 开发
  - 运维
---
Windows Server操作系统默认的安全策略，是不允许运行未签名的powershell代码程序。因为powershell脚本是由纯文本构成，很容易被修改和冒名顶替。为了不修改系统默认的安全策略而运行powershell代码，则需要对编写的脚本进行数字签名，提高安全性，确保不被修改。
  
如果要调整验证的策略，看通过下面的命令完成：

<pre class="prettyprint linenums"># 验证所有的脚本的签名信息，验证不通过，拒绝执行。
set-executionpolicy AllSigned
#从网络上下载的脚本执行会提示需要签名
set-executionpolicy remotesigned
</pre>

&nbsp;

## 生成代码签名数字证书

对脚本和代码签名，首选需要有数字证书。一般可以从两种渠道获取签名的证书：

1.从微软信任的第三方数字证书办法机构获取代码签名的数字证书。如：Versign、Globalsign、Symantec等数字证书办法机构。此类证书可官方用于对32 位或64 位用户模式（.exe、.cab、.dll、.ocx、.msi、.xpi 和 .xap 文件）和内核模式软件进行数字签名。优点：被微软操作系统直接信任，一次签名可运行在所有windows操作系统。

2.自签名证书。使用自建的CA或makecert.exe生成自签名的证书，此类签名不能直接被操作系统信任，不可用于Windows系统底层内核和驱动签名。

本文将使用Microsoft提供的makecert.exe程序创建自签名的代码签名证书。makecert.exe不能单独下载，但是它包含在.NET Framework和开发工具中。因此需要安装Virual Studio或Windows SDK。

<pre class="prettyprint linenums">#Windows Software Development Kit (SDK) for Windows 8.1
http://download.microsoft.com/download/B/0/C/B0C80BA3-8AD6-4958-810B-6882485230B5/standalonesdk/sdksetup.exe
#Microsoft Windows SDK for Windows 7 and .NET Framework 4
http://download.microsoft.com/download/A/6/A/A6AC035D-DA3F-4F0C-ADA4-37C8E5D34E3D/winsdk_web.exe
</pre>

配置环境变量，将SDK安装目录加入系统PATH，或直接进入该目录运行makecert.exe

<pre class="prettyprint linenums">#x64
C:\Program Files (x86)\Windows Kits\8.1\bin\x64
#x86
C:\Program Files (x86)\Windows Kits\8.1\bin\x86
</pre>

生成代码签名根证书，为了保护证书的安全性，对根证书使用密码保护。命令执行完成后，将会在当前目录创建根证书公钥和私钥。

<pre class="prettyprint linenums">makecert -n "CN=CodeSign Certificate Root" -a sha1 -eku 1.3.6.1.5.5.7.3.3 -r -sv Root.pvk Root.cer -ss Root -sr localMachine
</pre>

从根证书生成代码签名的用户证书。

<pre class="prettyprint linenums">#cn=Powershell CodeSign，可自行修改。
#-eku 参数：1.3.6.1.5.5.7.3.3，不可修改否则证书的预期目的属性就不是代码签名了。
#-ss "my"：证书存储位置，为本地用户
makecert -pe -n "CN=PowerShell User" -ss "my" -a sha1 -eku 1.3.6.1.5.5.7.3.3 -iv Root.pvk -ic Root.cer
#查看上面生成的证书
ls cert:\CurrentUser\My
</pre>

## 为Powershell脚本签名

获取用于签名的证书对象，$certificate，并使用 Set-AuthenticodeSignature 命名对代码进行签名

<pre class="prettyprint linenums">$certificate=ls cert:\CurrentUser\My | where {$_.subject -eq "CN=Powershell User"}
# 使用Set-AuthenticodeSignature 对代码进行签名
PS D:\&gt;&gt; Set-AuthenticodeSignature .\helloword.ps1 $certificate
    目录: D:\
SignerCertificate                         Status                                 Path
-----------------                         ------                                 ----
03A98605E2375FDBAF388ECAE0B323EFD2E04737  Valid                                  helloword.ps1
</pre>

成功签名后，会自动将签名的证书信息附加到文件的底部。

批量递归签名，对当前目录下所有扩展名为.ps1的代码进行签名。

<pre class="prettyprint linenums">Set-AuthenticodeSignature (ls *.ps1) $certificate
</pre>

[<img class="aligncenter size-full wp-image-309" src="http://www.wanglijie.cn/wp-content/uploads/2015/06/powershellcodesign1.jpg" alt="powershellcodesign" width="715" height="481" srcset="http://www.wanglijie.cn/wp-content/uploads/2015/06/powershellcodesign1.jpg 715w, http://www.wanglijie.cn/wp-content/uploads/2015/06/powershellcodesign1-300x202.jpg 300w" sizes="(max-width: 715px) 100vw, 715px" />](http://www.wanglijie.cn/wp-content/uploads/2015/06/powershellcodesign1.jpg)

## Powershell脚本签名验证

数字签名后的代码可以进行验证。可以手动验证或自动验证。签名验证会告诉你脚本是否信任，或者是否包含了恶意篡改。
  
用户自行验证：手动验证，可以检查一个脚本是否包含签名代码，签名者是谁？该签名者是否受信任。
  
自动验证：如果你将Powershell的脚本执行策略设置为AllSigned. Powershell会在你尝试运行脚本时自动验证，代码和脚本签名是否一致。并且会询问签名者是否受信任。

1.手动验证
  
Get-AuthenticodeSignature命令可以验证签名。例如创建一个脚本，不进行签名，通过该命令进行验证。属性StatusMessage会告诉你签名验证的结果。

<pre class="prettyprint linenums">#数字签名后未修改，则验证通过
PS D:\&gt; $checkResult=Get-AuthenticodeSignature .\helloworld.ps1
PS D:\&gt; $checkResult.Status
Valid
#对脚本进行简单修改后，就会提示“文件的哈希码和存储的签名不匹配”
PS D:\&gt; $checkResult=Get-AuthenticodeSignature .\helloworld.ps1
PS D:\&gt; $checkResult.Status
HashMismatch
</pre>

验证提示信息说明：

<table style="height: 338px;" width="742">
  <tr>
    <td width="181">
      成员名称
    </td>
    
    <td width="353">
      描述
    </td>
  </tr>
  
  <tr>
    <td width="181">
      HashMismatch
    </td>
    
    <td width="353">
      文件的哈希码和存储的签名不匹配
    </td>
  </tr>
  
  <tr>
    <td width="181">
      Incompatible
    </td>
    
    <td width="353">
      无法验证签名，因为与当前操作系统不兼容
    </td>
  </tr>
  
  <tr>
    <td width="181">
      NotSigned
    </td>
    
    <td width="353">
      文件没有签名
    </td>
  </tr>
  
  <tr>
    <td width="181">
      NotSupportedFileFormat
    </td>
    
    <td width="353">
      指定的文件格式不支持的系统签名。这通常意味着系统不知道如何签名或验证文件的类型。
    </td>
  </tr>
  
  <tr>
    <td width="181">
      NotTrusted
    </td>
    
    <td width="353">
      证书的发布者在系统中不受信任.
    </td>
  </tr>
  
  <tr>
    <td width="181">
      UnknownError
    </td>
    
    <td width="353">
      文件签名无效
    </td>
  </tr>
  
  <tr>
    <td width="181">
      Valid
    </td>
    
    <td width="353">
      该文件有一个有效的签名。这意味着只有签名的语法上是合法的。这并不意味着信任。
    </td>
  </tr>
</table>

2.自动验证

当你运行一个脚本时，Powershell会自动验证。即使验证过的脚本，如果有部分内容更新，自动验证也会给出警告。
  
在用户将脚本执行策略设置为AllSigned和RemoteSigned时，自动验证就会激活，如果将执行策略设置为AllSigned，所有的脚本都会验证。如果你选择RemoteSigned，从网络上下载的脚本执行会提示需要签名。

<pre class="prettyprint linenums"># 设置 ExecutionPolicy 为 AllSigned. 所有脚本必须有正确的签名
Set-ExecutionPolicy AllSigned
</pre>

运行未签名的脚本，系统会拒绝执行，如下所示：

<pre class="prettyprint linenums">PS D:\&gt; .\firstSignScript.ps1
.\firstSignScript.ps1 : 无法加载文件 D:\firstSignScript.ps1。文件 D:\firstSignScript.ps1 未进行数字签名。不会在系统上执
行该脚本。有关详细信息，请参阅 http://go.microsoft.com/fwlink/?LinkID=135170 中的 about_Execution_Policies。
所在位置 行:1 字符: 1
+ .\firstSignScript.ps1
+ ~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : SecurityError: (:) []，PSSecurityException
    + FullyQualifiedErrorId : UnauthorizedAccess
</pre>

数字签名后的脚本，仍然需要选择是否继续执行：

<pre class="prettyprint linenums">PS D:\&gt; .\firstSignScript.ps1

是否要运行来自此不可信发布者的软件?
文件 D:\firstSignScript.ps1 由 CN=Pactera EDS Powershell CodeSign
发布，该文件对于你的系统是不可信的。请只运行来自可信发布者的脚本。
[V] 从不运行(V)  [D] 不运行(D)  [R] 运行一次(R)  [A] 始终运行(A)  [?] 帮助 (默认值为“D”): a
我的第一个签名脚本
</pre>

## 在其他服务器运行签名的代码

在本地计算机签名的代码无法运行在其他的计算机上，需要将该签名的根证书（公钥）、签名证书（公钥和私钥）导入到目标服务器。并将根证书导入到“受信任的根证书办法机构”即可正常运行签名后的代码。

转载请注明：[自动化运维](http://www.wanglijie.cn) &raquo; [自签名证书对Powershell代码签名](http://www.wanglijie.cn/2015/06/%e8%87%aa%e7%ad%be%e5%90%8d%e8%af%81%e4%b9%a6%e5%af%b9powershell%e4%bb%a3%e7%a0%81%e7%ad%be%e5%90%8d.html)