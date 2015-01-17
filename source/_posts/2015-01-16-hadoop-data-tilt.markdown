---
layout: post
title: "Hadoop数据倾斜问题总结"
date: 2015-01-16 09:41:26 +0800
comments: true
categories: Hadoop
keywords: Hadoop数据倾斜问题
description: Hadoop数据倾斜问题
---
首先解释什么是数据倾斜问题，举一个例子，图书馆有A0、A1、A2...A9共10个书架，A0书架有1000本书，而A1到A9书架有100本书，现在要统计图书的总数量，这时我们会发现A1到A9很快就统计完了，而A0书架需要统计很长时间，显然A0成为整个统计过程的瓶颈。  

Hadoop最根本的思想就是分而治之，数据倾斜会导致分布式的计算效率依赖单个节点，这与它的初衷是背道而驰的，但是Hadoop又没有从根本层面解决这个问题，所以需要我们在业务设计和开发时来解决这个问题，最好的解决方法是设计时避免让这个问题出现。如果无法避免，那么解决数据倾斜问题和核心思想只有一个：让map输出的key均匀分布到各个reduce中去（mapjoin例外，直接在内存中解决，牺牲内存空间提高效率，只适用于较小的输入数据）  

产生数据倾斜的具体原因以及解决方案可以参考阿里的这篇博客：http://www.alidata.org/archives/2109  

本文主要想总结一下各种处理方案的思路：

<!--more-->

### 1、Distributed Cache ###
Distributed Cache是Hadoop提供的文件缓存工具，会自动将需要缓存的文件分发到各个DataNode上去，对于key分布不均匀的数据并且数据量不大，可以使用Distributed Cache将文件缓存到各个节点上，这样在做map的时候直接从内存中读取数据不需要到reduce阶段处理从而提高性能。

创建job时可以添加需要cache的文件  
	Job job = new Job();
	...
	job.addCacheFile(new Path(cacheFilename).toUri());

在mapper端使用  
	Path[] localPaths = context.getLocalCacheFiles();
	...

### 2、Map Join ###
如果是Hive，就是用Map Join，底层用的就是Distributed Cache，
	SELECT /*+ MAPJOIN(b) */ a.key, a.value
	FROM a JOIN b ON a.key = b.key
在hive0.7之后，可以不用显示的指定MAPJOIN关键词，hive会根据参数配置来决定是否需要用到mapjoin
	hive.auto.convert.join=true//默认打开mapjoin优化
	hive.mapjoin.smalltable.filesize=25000000//文件达到多大时开启mapjoin优化，阀值控制


### 3、转换map输出的key值###
如果我们无法在设计时避免，也无法在map时处理，那只能在最后一刻来弥补，比如map输出的key大量为空，那么可以用一个随机数来代替空值，这样数据就是均摊到多个reduce上了

### 4、步骤拆解 ###
将一步操作拆解为多步，将倾斜的数据单独拿出来处理，最后再与正常数据的结果合并就可以了，具体实现可以看这篇博客：http://www.alidata.org/archives/2109  最后的总结

### 5、参数调整 ###
	hive.map.aggr = true//map端部分聚合，相当于Combiner
	hive.groupby.skewindata=true//有数据倾斜的时候自动进行负载均衡

如果大家以后遇到数据倾斜的问题，可以尝试从这几个方面来想办法优化。

