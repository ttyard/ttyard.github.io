---
id: 398
title: AWS修复Windows远程桌面无法访问，防火墙配置错误问题
date: 2015-09-15T14:04:06+00:00
author: 深海游鱼
layout: post
guid: http://www.wanglijie.cn/?p=398
permalink: /2015/09/398.html
views:
  - "357"
image: /wp-content/uploads/2015/09/aws-recovery-1-220x150.png
categories:
  - 云计算
tags:  
  - 公有云运维  
---
我们有时候Amazon AWS的云服务器，由于AWS没有提供类似国内云服务器厂商的远程控制台服务，在出现网络、防火墙和远程桌面服务问题导致云服务器无法访问的情况发生后，进行恢复就特别的麻烦。
  
所以，在涉及远程桌面、防火墙配置的事后要格外的谨慎，否则就麻烦了。为了修复无法访问的实例，只能通过将故障服务器系统磁盘挂载到另外的虚拟机中，修改注册表来修改/禁用防火墙配置。

## 1、停止故障实例

<p style="text-align: center;">
  将无法访问的服务器实例关机，断开其根卷（即将系统盘从原系统中取消挂载）。<br /> <a href="http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-1.png"><img class="aligncenter size-full wp-image-399" src="http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-1.png" alt="aws-recovery-1" width="391" height="520" srcset="http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-1.png 391w, http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-1-226x300.png 226w" sizes="(max-width: 391px) 100vw, 391px" /></a><br /> 图1-停止实例<br /> <a href="http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-2.png"><img class="aligncenter size-full wp-image-400" src="http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-2.png" alt="aws-recovery-2" width="720" height="499" srcset="http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-2.png 720w, http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-2-300x208.png 300w" sizes="(max-width: 720px) 100vw, 720px" /></a><br /> 图2-断开卷<a href="http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-4.png"><br /> </a>
</p>

## 2、在受影响的实例所在的可用区中启动临时实例

临时实例必须与原实例在同一区域，如:cn-north-1a或cn-north-1b，否则无法挂载。
  
如果您的临时实例与原始实例基于相同的 AMI(即同一个系统镜像)，且操作系统版本高于 Windows Server 2003，则您必须修改故障系统系统卷的磁盘签名，否则在恢复原始实例的根卷之后，由于磁盘签名冲突，您将无法启动原始实例。 相同AMI挂载恢复方法，见附录1.
  
在此，我们选择与故障系统不同AMI系统镜像来挂载原始卷。例如，如果原始实例使用 Windows Server 2008 R2 的 AWS Windows AMI，则使用 Windows Server 2012 或 Windows Server 2003 的 AWS Windows AMI 来启动临时实例。（要查找适用于 Windows Server 2003 的 AMI，请使用名称Windows\_Server-2003-R2\_SP2 来搜索 AMI。）

## 3.将根卷从受影响的实例连接到此临时实例。

连接到临时实例，打开 Disk Management (磁盘管理) 实用工具，将驱动器联机。

## 4.注册表管理器加载故障卷SYSTEM注册表文件

打开 Regedit，选择 HKEY\_LOCAL\_MACHINE。从 File (文件) 菜单，单击 Load Hive (加载 Hive)。选择驱动器，打开文件 Windows\System32\config\SYSTEM，在出现提示时指定键名（您可以使用任何名称）。
  
[<img class="aligncenter size-full wp-image-401" src="http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-3.png" alt="aws-recovery-3" width="927" height="518" srcset="http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-3.png 927w, http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-3-300x168.png 300w" sizes="(max-width: 927px) 100vw, 927px" />](http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-3.png)

<p style="text-align: center;">
  图3-选择注册表文件
</p>

[<img class="aligncenter size-full wp-image-402" src="http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-4.png" alt="aws-recovery-4" width="596" height="285" srcset="http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-4.png 596w, http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-4-300x143.png 300w" sizes="(max-width: 596px) 100vw, 596px" />](http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-4.png)

<p style="text-align: center;">
  图4-输入加载的注册表名称
</p>

## 5.修改注册表，禁用防火墙相关条目

选择刚加载的键并导航至ControlSet001\Services\SharedAccess\Parameters\FirewallPolicy。对于名称格式为 xxxxProfile 的每个键，选择键，将 EnableFirewall 从 1 更改为 0。再次选择该键，从 File (文件) 菜单中单击 Unload Hive (卸载 Hive)。
  
[<img class="aligncenter size-full wp-image-403" src="http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-5.png" alt="aws-recovery-5" width="792" height="436" srcset="http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-5.png 792w, http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-5-300x165.png 300w" sizes="(max-width: 792px) 100vw, 792px" />](http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-5.png)

<p style="text-align: center;">
  图5-修改注册表，禁用防火墙
</p>

或者，你们可以找到配置错误的防火墙记录，禁用该条目或修改成正确的数值，导航至:ControlSet001\Services\SharedAccess\Parameters\FirewallPolicy\FirewallRules
  
[<img class="aligncenter size-full wp-image-404" src="http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-6.png" alt="aws-recovery-6" width="768" height="661" srcset="http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-6.png 768w, http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-6-300x258.png 300w" sizes="(max-width: 768px) 100vw, 768px" />](http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-6.png)

<p style="text-align: center;">
  图6-修改单条防火墙记录
</p>

完成后，选中挂载的注册表文件，选择“文件”菜单，“Unload Hive”来卸载Hive。
  
[<img class="aligncenter size-full wp-image-405" src="http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-7.png" alt="aws-recovery-7" width="376" height="320" srcset="http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-7.png 376w, http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-7-300x255.png 300w" sizes="(max-width: 376px) 100vw, 376px" />](http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-7.png)

<p style="text-align: center;">
  图7-卸载加载的注册表文件
</p>

## 6. 将该卷从临时实例分离。如果您不再使用临时实例，则可以将其终止。

## 7 通过将受影响的实例的根卷挂载为 /dev/sda1 将其还原。

## 8. 启动实例。

## 附录1 ：

本节内容是针对相同AMI系统镜像挂载故障磁盘（卷）并修复的方法，由于Windows 2003以上版本系统新增了磁盘的数字签名，若磁盘签名ID不匹配将会导致修改后的磁盘无法在原始虚拟机中挂载将无法正常的启动，要解决这个问题需要在挂载前对原始磁盘的数字签名ID和磁盘BCD中的ID匹配文件进行修改。

### A.

打开注册表程序regedit.exe。从&#8221;File (文件)” 菜单，单击“ Load Hive (加载 Hive)”，选择挂载的原始根卷（系统盘,当前为D盘），依次选择：D:\boot\bcd。
  
系统文件Boot文件夹及下面的文件都是系统隐藏属性，首选取消相关文件保护选项才可以看到。
  
[<img class="aligncenter size-full wp-image-406" src="http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-8.png" alt="aws-recovery-8" width="917" height="509" srcset="http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-8.png 917w, http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-8-300x167.png 300w" sizes="(max-width: 917px) 100vw, 917px" />](http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-8.png)

<p style="text-align: center;">
  图8-使用注册表管理器加载BCD文件
</p>

### B.

在加载的BCD注册表文件中，选择“编辑”&#8211;“搜索”，查找“Windows Boot Manager”字符串，可在名为 12000004 的注册表项下找到匹配的项。
  
[<img class="aligncenter size-full wp-image-407" src="http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-9.png" alt="aws-recovery-9" width="769" height="372" srcset="http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-9.png 769w, http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-9-300x145.png 300w" sizes="(max-width: 769px) 100vw, 769px" />](http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-9.png)

<p style="text-align: center;">
  图9-查找指定字符串
</p>

### C.

选择与在上一步骤中找到的注册表项同级的名为 11000001 的项。查看 Element 值的数据。在数据中的偏移 0x38 处查找四字节磁盘签名。数据95C39193表示的磁盘签名.
  
[<img class="aligncenter size-full wp-image-408" src="http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-10.png" alt="aws-recovery-10" width="835" height="398" srcset="http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-10.png 835w, http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-10-300x143.png 300w" sizes="(max-width: 835px) 100vw, 835px" />](http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-10.png)

<p style="text-align: center;">
  图10-查看磁盘签名ID
</p>

### D.

首先修改注册表项11000001的当前用户读写权限，否则修改数据后无法保存。鼠标右键选择注册表项&#8221;11000001&#8243;&#8211;>&#8221;权限&#8221;
  
[<img class="aligncenter size-full wp-image-409" src="http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-11.png" alt="aws-recovery-11" width="604" height="422" srcset="http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-11.png 604w, http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-11-300x210.png 300w" sizes="(max-width: 604px) 100vw, 604px" />](http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-11.png)

<p style="text-align: center;">
  图11-修改条目读写权限
</p>

### E.

颠倒&#8221;11000001&#8243;->&#8221;Element&#8221;这些字节以创建磁盘签名并将其记下。例如，将原来的95C39193修改成9391C395。
  
[<img class="aligncenter size-full wp-image-410" src="http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-12.png" alt="aws-recovery-12" width="651" height="349" srcset="http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-12.png 651w, http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-12-300x161.png 300w" sizes="(max-width: 651px) 100vw, 651px" />](http://images.wanglijie.cn/public/img/posts/2015/09/aws-recovery-12.png)

<p style="text-align: center;">
  图12-修改签名ID
</p>

注意：该注册表项下有多个Element值，一定需要选择&#8221;Windows Boot Manager&#8221;上层目录的&#8221;11000001&#8243;.

### F.

在命令提示符窗口中，运行以下命令以启动 Microsoft DiskPart。

<pre class="prettyprint linenums">C:\&gt; diskpart
</pre>

### G.

使用list disk 查看挂载的磁盘ID，并选择挂载的故障卷磁盘。

<pre class="prettyprint linenums">DISKPART&gt; list disk
磁盘 ### 状态 大小 可用 Dyn Gpt
-------- ------------- ------- ------- --- ---
磁盘 0 联机 298 GB 3072 KB
* 磁盘 1 联机 111 GB 1024 KB
磁盘 2 联机 238 MB 0 B
磁盘 3 联机 149 GB 1024 KB
</pre>

### H.

运行以下 DiskPart 命令以选择卷。（您可以使用 Disk Management (磁盘管理) 实用工具来验证磁盘编号为 1。）

<pre class="prettyprint linenums">DISKPART&gt; select disk 1

Disk 1 is now the selected disk.
</pre>

### I.

查看磁盘签名，运行以下 DiskPart 命令以获取磁盘签名

<pre class="prettyprint linenums">DISKPART&gt;uniqueid disk
磁盘 ID: 95C39193
</pre>

### J.

如果上一步骤中显示的磁盘签名与前面记下的 BCD 中的磁盘签名不匹配，请使用以下 DiskPart 命令更改磁盘签名以使其匹配：

<pre class="prettyprint linenums">DISKPART&gt; uniqueid disk id=9391C395
</pre>

### K.

使用 Disk Management (磁盘管理) 实用工具，将驱动器脱机。
  
Note
  
如果临时实例运行的操作系统与受影响实例相同，则驱动器将自动脱机，因此您无需手动使其脱机。

### L.

执行上面的步骤6关闭实例，解除关联并挂回原故障系统即可成功启动。

其他故障修复方案，请参考官方手册：
  
http://docs.amazonaws.cn/AWSEC2/latest/WindowsGuide/troubleshooting-windows-instances.html

转载请注明：[自动化运维](http://www.wanglijie.cn) &raquo; [AWS修复Windows远程桌面无法访问，防火墙配置错误问题](http://www.wanglijie.cn/2015/09/398.html)