---
id: 290
title: Windows SQL Server 性能计数器详细说明
date: 2015-06-05T09:09:54+00:00
author: 深海游鱼
layout: post
guid: http://www.wanglijie.cn/?p=290
permalink: '/2015/06/windows-sql-server-%e6%80%a7%e8%83%bd%e8%ae%a1%e6%95%b0%e5%99%a8%e8%af%a6%e7%bb%86%e8%af%b4%e6%98%8e.html'
views:
  - "344"
categories:
  - 运维
tags:
  - Windows  
---
Windows SQL Server 性能计数器详细说明

<table width="575">
  <tr>
    <td colspan="3" width="575">
      SQL Server性能计数器
    </td>
  </tr>
  
  <tr>
    <td width="140">
      Performance Object
    </td>
    
    <td width="187">
      Counter
    </td>
    
    <td width="248">
      Description
    </td>
  </tr>
  
  <tr>
    <td rowspan="2" width="140">
      Processor
    </td>
    
    <td width="187">
      %processor Time
    </td>
    
    <td width="248">
      指处理器执行非闲置线程时间的百分比,测量处理器繁忙的时间 这个计数器设计成用来作为处理器活动的主要指示器,可以选择单个CPU实例,也可以选择Total
    </td>
  </tr>
  
  <tr>
    <td width="187">
      Interrupts/sec
    </td>
    
    <td width="248">
      处理器正在处理的来自应用程序或硬件的中断的数量
    </td>
  </tr>
  
  <tr>
    <td rowspan="17" width="140">
      PhysicalDisk
    </td>
    
    <td rowspan="13" width="187">
      % Disk Time
    </td>
    
    <td width="248">
      计数器监视磁盘忙于读/写活动所用时间的百分比.在系统监视器中，PhysicalDisk: % Disk Time 计数器监视磁盘忙于读/写活动所用时间的百分比。如果 PhysicalDisk: % Disk Time 计数器的值较高（大于 90%），请检查 PhysicalDisk: Current Disk Queue Length 计数器了解等待进行磁盘访问的系统请求数量。等待 I/O 请求的数量应该保持在不超过组成物理磁盘的轴数的 1.5 到 2 倍。大多数磁盘只有一个轴，但独立磁盘冗余阵列 (RAID) 设备通常有多个轴。硬件 RAID 设备在系统监视器中显示为一个物理磁盘。通过软件创建的多个 RAID 设备在系统监视器中显示为多个实例。
    </td>
  </tr>
  
  <tr>
    <td width="248">
      可以使用 Current Disk Queue Length 和 % Disk Time 计数器的值检测磁盘子系统中的瓶颈。如果 Current Disk Queue Length 和 % Disk Time 计数器的值一直很高，则考虑下列事项：
    </td>
  </tr>
  
  <tr>
    <td width="248">
      1.使用速度更快的磁盘驱动器。
    </td>
  </tr>
  
  <tr>
    <td width="248">
      2.将某些文件移至其他磁盘或服务器。
    </td>
  </tr>
  
  <tr>
    <td width="248">
      3.如果正在使用一个 RAID 阵列，则在该阵列中添加磁盘。
    </td>
  </tr>
  
  <tr>
    <td width="248">
      计数器监视磁盘忙于读/写活动所用时间的百分比.在系统监视器中，PhysicalDisk: % Disk Time 计数器监视磁盘忙于读/写活动所用时间的百分比。
    </td>
  </tr>
  
  <tr>
    <td width="248">
      如果 PhysicalDisk: % Disk Time 计数器的值较高（大于 90%），请检查PhysicalDisk: Current Disk Queue Length 计数器了解等待进行磁
    </td>
  </tr>
  
  <tr>
    <td width="248">
      盘访问的系统请求数量。等待 I/O 请求的数量应该保持在不超过组成物理磁盘的轴数的1.5 到 2 倍。大多数磁盘只有一个轴，但独立磁盘冗余阵列
    </td>
  </tr>
  
  <tr>
    <td width="248">
      (RAID) 设备通常有多个轴。硬件 RAID 设备在系统监视器中显示为一个物理磁盘。通过软件创建的多个 RAID 设备在系统监视器中显示为多个实例。
    </td>
  </tr>
  
  <tr>
    <td width="248">
      可以使用 Current Disk Queue Length 和 % Disk Time 计数器的值检测磁盘子系统中的瓶颈。如果 Current Disk Queue Length 和 % Disk Time 计数器的值一直很高，则考虑下列事项：
    </td>
  </tr>
  
  <tr>
    <td width="248">
      1.使用速度更快的磁盘驱动器。
    </td>
  </tr>
  
  <tr>
    <td width="248">
      2.将某些文件移至其他磁盘或服务器。
    </td>
  </tr>
  
  <tr>
    <td width="248">
      3.如果正在使用一个 RAID 阵列，则在该阵列中添加磁盘。
    </td>
  </tr>
  
  <tr>
    <td width="187">
      Avg.Disk Queue Length
    </td>
    
    <td width="248">
      指读取和写入请求(为所选磁盘在实例间隔中列队的)的平均数
    </td>
  </tr>
  
  <tr>
    <td width="187">
      Current Disk Queue Length
    </td>
    
    <td width="248">
      指示被挂起的磁盘 I/O 请求的数量。如果这个值始终高于 2，就表示产生了拥塞
    </td>
  </tr>
  
  <tr>
    <td width="187">
      Avg.Disk Bytes/Transfer
    </td>
    
    <td width="248">
      写入或读取操作时向磁盘传送或从磁盘传出字节的平均数
    </td>
  </tr>
  
  <tr>
    <td width="187">
      Disk Bytes/sec
    </td>
    
    <td width="248">
      在读写操作中，从磁盘传出或传送到磁盘的字节速率
    </td>
  </tr>
  
  <tr>
    <td rowspan="5" width="140">
      Memory
    </td>
    
    <td width="187">
      Pages/sec
    </td>
    
    <td width="248">
      被请求页面的数量.
    </td>
  </tr>
  
  <tr>
    <td width="187">
      Available Bytes
    </td>
    
    <td width="248">
      可用物理内存的数量
    </td>
  </tr>
  
  <tr>
    <td width="187">
      Committed Bytes
    </td>
    
    <td width="248">
      已分配给物理 RAM 用于存储或分配给页面文件的虚拟内存
    </td>
  </tr>
  
  <tr>
    <td width="187">
      Pool Nonpaged Bytes
    </td>
    
    <td width="248">
      未分页池系统内存区域中的 RAM 数量
    </td>
  </tr>
  
  <tr>
    <td width="187">
      Page Faults/sec
    </td>
    
    <td width="248">
      是每秒钟出错页面的平均数量
    </td>
  </tr>
  
  <tr>
    <td rowspan="3" width="140">
      Network Interface
    </td>
    
    <td width="187">
      Bytes Received/sec
    </td>
    
    <td width="248">
      使用本网络适配器接收的字节数
    </td>
  </tr>
  
  <tr>
    <td width="187">
      Bytes Sent/sec
    </td>
    
    <td width="248">
      使用本网络适配器发送的字节数
    </td>
  </tr>
  
  <tr>
    <td width="187">
      Bytes Total/sec
    </td>
    
    <td width="248">
      使用本网络适配器发送和接收的字节数
    </td>
  </tr>
  
  <tr>
    <td width="140">
      Server
    </td>
    
    <td width="187">
      Bytes Received/sec
    </td>
    
    <td width="248">
      把此计数器与网络适配器的总带宽相比较，确定网络连接是否产生瓶颈
    </td>
  </tr>
  
  <tr>
    <td rowspan="3" width="140">
      SQL Server Access Methods
    </td>
    
    <td width="187">
      Page Splits/sec
    </td>
    
    <td width="248">
      每秒由于索引页溢出而发生的页拆分数.如果发现页分裂的次数很多,考虑提高Index的填充因子.数据页将会有更多的空间保留用于做数据的填充,从而减少页拆分
    </td>
  </tr>
  
  <tr>
    <td width="187">
      Pages Allocated/sec
    </td>
    
    <td width="248">
      在此 SQL Server 实例的所有数据库中每秒分配的页数。这些页包括从混合区和统一区中分配的页
    </td>
  </tr>
  
  <tr>
    <td width="187">
      Full Scans/sec
    </td>
    
    <td width="248">
      每秒不受限制的完全扫描数. 这些扫描可以是基表扫描，也可以是全文索引扫描
    </td>
  </tr>
  
  <tr>
    <td rowspan="6" width="140">
      SQL Server: SQL Statistics
    </td>
    
    <td rowspan="2" width="187">
      Batch Requests/Sec
    </td>
    
    <td width="248">
      每秒收到的 Transact-SQL 命令批数。这一统计信息受所有约束（如 I/O、用户数、高速缓存大小、请求的复杂程度等）影响。
    </td>
  </tr>
  
  <tr>
    <td width="248">
      批处理请求数值高意味着吞吐量
    </td>
  </tr>
  
  <tr>
    <td rowspan="2" width="187">
      SQL Compilations/Sec
    </td>
    
    <td width="248">
      每秒的编译数。表示编译代码路径被进入的次数。包括 SQL Server 中语句级重新编译导致的编译。当 SQL Server 用户活动稳定后，
    </td>
  </tr>
  
  <tr>
    <td width="248">
      该值将达到稳定状态
    </td>
  </tr>
  
  <tr>
    <td rowspan="2" width="187">
      Re-Compilations/Sec
    </td>
    
    <td width="248">
      每秒语句重新编译的次数。计算语句重新编译被触发的次数。一般来说，这个数最好较小,存储过程在理想情况下应该只编译一次，
    </td>
  </tr>
  
  <tr>
    <td width="248">
      然后执行计划被重复使用. 如果该计数器的值较高，或许需要换个方式编写存储过程，从而减少重编译的次数
    </td>
  </tr>
  
  <tr>
    <td rowspan="4" width="140">
      SQL Server: Databases
    </td>
    
    <td width="187">
      Log Flushes/sec
    </td>
    
    <td width="248">
      每秒日志刷新数目
    </td>
  </tr>
  
  <tr>
    <td width="187">
      Active Transactions
    </td>
    
    <td width="248">
      数据库的活动事务数
    </td>
  </tr>
  
  <tr>
    <td rowspan="2" width="187">
      Backup/Restore Throughput/sec
    </td>
    
    <td width="248">
      每秒数据库的备份和还原操作的读取/写入吞吐量。例如，并行使用多个备份设备或使用更快的设备时,可以测量数据库备份操作性能的变化情况。
    </td>
  </tr>
  
  <tr>
    <td width="248">
      数据库的备份或还原操作的吞吐量可以确定备份和还原操作的进程和性能
    </td>
  </tr>
  
  <tr>
    <td rowspan="3" width="140">
      SQL Server General Statistics
    </td>
    
    <td width="187">
      User Connections
    </td>
    
    <td width="248">
      系统中活动的SQL连接数. 该计数器的信息可以用于找出系统的最大并发用户数
    </td>
  </tr>
  
  <tr>
    <td width="187">
      Temp Tables Creation Rate
    </td>
    
    <td width="248">
      每秒创建的临时表/表变量的数目
    </td>
  </tr>
  
  <tr>
    <td width="187">
      Temp Tables For Destruction
    </td>
    
    <td width="248">
      等待被清除系统线程破坏的临时表/表变量数
    </td>
  </tr>
  
  <tr>
    <td rowspan="3" width="140">
      SQL Server Locks
    </td>
    
    <td width="187">
      Number of Deadlocks/sec
    </td>
    
    <td width="248">
      指每秒导致死锁的锁请求数. 死锁对于应用程序的可伸缩性非常有害, 并且会导致恶劣的用户体验. 该计数器必须为0
    </td>
  </tr>
  
  <tr>
    <td width="187">
      Average Wait Time (ms)
    </td>
    
    <td width="248">
      每个导致等待的锁请求的平均等待时间
    </td>
  </tr>
  
  <tr>
    <td width="187">
      Lock requests/sec
    </td>
    
    <td width="248">
      锁管理器每秒请求的新锁和锁转换数. 通过优化查询来减少读取次数, 可以减少该计数器的值
    </td>
  </tr>
  
  <tr>
    <td rowspan="4" width="140">
      SQL Server:Memory Manager
    </td>
    
    <td width="187">
      Total Server Memory (KB)
    </td>
    
    <td width="248">
      从缓冲池提交的内存(这不是 SQL Server 使用的总内存)
    </td>
  </tr>
  
  <tr>
    <td width="187">
      Target Server Memory (KB)
    </td>
    
    <td width="248">
      服务器能够使用的动态内存总量
    </td>
  </tr>
  
  <tr>
    <td width="187">
      SQL Cache Memory(KB)
    </td>
    
    <td width="248">
      服务器正在用于动态 SQL 高速缓存的动态内存总数
    </td>
  </tr>
  
  <tr>
    <td width="187">
      Memory Grants Pending
    </td>
    
    <td width="248">
      指每秒等待工作空间内存授权的进程数. 该计数器应该尽可能接近0,否则预示可能存在着内存瓶颈
    </td>
  </tr>
  
  <tr>
    <td rowspan="6" width="140">
      SQL Server Buffer Manager
    </td>
    
    <td width="187">
      Buffer Cache Hit Ratio
    </td>
    
    <td width="248">
      缓存命中率,在缓冲区高速缓存中找到而不需要从磁盘中读取(物理I/O)的页的百分比. 如果该值较低则可能存在内存不足或不正确的索引
    </td>
  </tr>
  
  <tr>
    <td width="187">
      Page Reads/sec
    </td>
    
    <td width="248">
      每秒发出的物理数据库页读取数。此统计信息显示的是所有数据库间的物理页读取总数。由于物理 I/O 的开销大，可以通过使用更大的数据缓存、智能索引、更有效的查询或更改数据库设计等方法，将开销降到最低
    </td>
  </tr>
  
  <tr>
    <td width="187">
      Page Writes/sec
    </td>
    
    <td width="248">
      每秒执行的物理数据库页写入数
    </td>
  </tr>
  
  <tr>
    <td width="187">
      Page Life Expectancy
    </td>
    
    <td width="248">
      页若不被引用将在缓冲池中停留的秒数
    </td>
  </tr>
  
  <tr>
    <td width="187">
      Lazy Writes/Sec
    </td>
    
    <td width="248">
      每秒被缓冲区管理器的惰性编写器写入的缓冲区数
    </td>
  </tr>
  
  <tr>
    <td width="187">
      Checkpoint Pages/Sec
    </td>
    
    <td width="248">
      由要求刷新所有脏页的检查点或其他操作每秒刷新到磁盘的页数
    </td>
  </tr>
</table>

转载请注明：[自动化运维](http://www.wanglijie.cn) &raquo; [Windows SQL Server 性能计数器详细说明](http://www.wanglijie.cn/2015/06/windows-sql-server-%e6%80%a7%e8%83%bd%e8%ae%a1%e6%95%b0%e5%99%a8%e8%af%a6%e7%bb%86%e8%af%b4%e6%98%8e.html)