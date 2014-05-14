---
layout: post
title: "Titan图数据库介绍"
date: 2014-05-14 22:35:18 +0800
comments: true
categories: 
---
Titan是一个高可用的分布式的图数据库，并且可以支撑上千个用户的并发事务，它有下面这些特性：

- 弹性和性能的线性扩展
- 容错性
- 多数据中心的高可用性和热备份
- 支持事务的ACID和最终一致性
- 支持多种不同的后端存储  
*Apache Cassandra（分布式）  
Apache HBase（分布式）  
Oracle BerkeleyDB（本地的）  
Persistit（本地）*  
- 支持多种后端索引    
*ElasticSearch  
Apache Lucene*
- 与图形处理栈TinkerPop原生集成     
	*图查询语言Gremlin  
	对象到图的映射器Frames  
	图服务器Rexster  
	标准图API：Blueprints*  
- Apache2 license 开源协议

Titan最大的优势在于其分布式和线性扩展，性能要高于Neo4j。还有支持HBase数据存储，这样可以和整个Hadoop平台完美结合起来，与YARN平台上面其它应用共享数据，但就这一点，以后Tian可能会代替Neo4j成为图数据库的主流。但是Titan目前应用还不是特别广泛，我们也是在尝试，最高版本是0.4，而且有很多需要改进的地方，包括与HBase的配置挺麻烦的，还无法放到YARN上来管理等等。

更多关于Titan的文档可以看[这里](https://github.com/thinkaurelius/titan/wiki)

可以从[这里](https://github.com/thinkaurelius/titan/wiki/Getting-Started)开始
