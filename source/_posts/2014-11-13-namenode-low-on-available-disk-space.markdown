---
layout: post
title: "Hadoop集群NameNode运行中进入安全模式的问题：NameNode low on available disk space"
date: 2014-11-13 15:23:26 +0800
comments: true
categories: Hadoop
keywords: Hadoop集群NameNode运行中进入安全模式的问题
description: Hadoop集群NameNode运行中进入安全模式的问题
---
问题描述：Hadoop集群在运行过程，早上来的时候发现NameNode自动进入安全模式，导致系统运行的MR任务失败。

最开始怀疑是HDFS的块复制没有达到最低的要求，通过hadoop fsck -blocks查看，Minimally replicated blocks为100%，说明blocks都达到了复制要求，最后查看了NameNode的日志发现几行警告信息：

    2014-11-12 21:31:38,478 WARN org.apache.hadoop.hdfs.server.namenode.NameNodeResourceChecker (org.apache.hadoop.hdfs.server.namenode.FSNamesystem$NameNodeResourceMonitor@1f1dd4b6): Space available o
    n volume 'null' is 13119488, which is below the configured reserved amount 104857600
    2014-11-12 21:31:38,478 WARN org.apache.hadoop.hdfs.server.namenode.FSNamesystem (org.apache.hadoop.hdfs.server.namenode.FSNamesystem$NameNodeResourceMonitor@1f1dd4b6): NameNode low on available di
    sk space. Entering safe mode.
    2014-11-12 21:31:38,478 INFO org.apache.hadoop.hdfs.StateChange (org.apache.hadoop.hdfs.server.namenode.FSNamesystem$NameNodeResourceMonitor@1f1dd4b6): STATE* Safe mode is ON. 
    Resources are low on NN. Please add or free up more resources then turn off safe mode manually. NOTE:  If you turn off safe mode before adding resources, the NN will immediately return to safe mode
    . Use "hdfs dfsadmin -safemode leave" to turn safe mode off.

通过日志分析发现这样的错误：`NameNode low on available disk space. Entering safe mode`  

在网上查到hadoop-jira上面有这样的记录：https://issues.apache.org/jira/browse/HDFS-4425  

另外这里还有详细的解释：https://support.pivotal.io/hc/en-us/articles/201455807-Namenode-logs-reports-Space-available-on-volume-null-is-below-threshold-and-enters-safe-mode  

原因是Hadoop的源码里面有一个类 `NameNodeResourceChecker` 负责检查NameNode的磁盘空间，如果磁盘空间低于100M则进入到安全模式，导致系统不可用，**那么问题来了**，是什么原因导致磁盘空间暴涨了呢？最后发现是HBase的DEBUG日志没有关闭导致日志文件非常大，最终磁盘空间占满，NameNode不可用。
