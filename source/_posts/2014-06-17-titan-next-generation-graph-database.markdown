---
layout: post
title: "Titan:下一代分布式图数据库"
date: 2014-06-17 10:23:30 +0800
comments: true
categories: Titan
keywords: Titan, 下一代分布式图数据库,Graph Database,Neo4j,分布式图形数据库
description: Titan,下一代分布式图数据库
---
[Titan](http://thinkaurelius.github.io/titan/)是一个由[Aurelius](http://thinkaurelius.com/)维护的开源协议为[Apache 2.0](http://www.apache.org/licenses/LICENSE-2.0.html)的分布式图形数据库。
<!--more-->
Graph Database作为NoSQL数据库四种存储模式(graph, key-value, column, and document)的其中一种，近年来发展迅猛，因为随着人工智能和社交网络不断发展和融合，数据结构越来越复杂，举个例子，以用户为中心的模型，用户的相关数据可能来源他的社交网络，也可能来源他的网购记录，也可能来源他的个人可穿戴设备等等，这个数据会呈现爆炸性增长，如果用户基数为千万级，再去做关联和流行度分析会非常复杂，Graph Database处理这样的需求具体天生的优势。目前市面上的[Graph Database](http://en.wikipedia.org/wiki/Graph_database#Graph_database_projects)有很多，[Neo4j](http://www.neo4j.org/)是最为成熟和知名度最高的产品，但因为Neo4j[不支持分片](http://stackoverflow.com/questions/21558589/neo4j-sharding-aspect/21566766#21566766)导致其存在可伸缩性的问题，但是貌似Neo4j已经推出相应的解决方案架构，参考这里[Neo4j HA-1](http://info.neotechnology.com/rs/neotechnology/images/Understanding%20Neo4j%20Scalability\(2\).pdf)、[Neo4j HA-2](http://neo4j.com/blog/2013-whats-coming-next-in-neo4j/)。

Titan作为新一代的Graph Database，还比较年轻，但非常有前途，它的优势有几方面：

- 天生支持分布式：横向扩展很容易，并且性能可以线性增长
- 性能：Titan官方在Titan-0.1-alpha做过一个[测试](http://thinkaurelius.com/2012/08/06/titan-provides-real-time-big-graph-data/)，性能表现非常强劲
- 后端存储无关：它可以将数据存储在不同的数据库，目前支持HBase、Cassandra和BerkeleyDB，而且Titan 0.5.0将会集成另外一个模块：Titan/Hadoop，这样会让Titan与现有的数据平台结合更加容易
- 后端索引无关：目前支持[ElasticSearch](http://www.elasticsearch.org/)和[Apache Lucene](http://lucene.apache.org/)两种索引
- 支持多数据中心的高可用和热备份
- 原生支持[tinkerpop](http://www.tinkerpop.com/)：[tinkerpop](http://www.tinkerpop.com/)是一系列Graph领域的开源软件栈

Titan的架构图：  
{% img /images/titan-next-1.png %} 

#### 总结 ####
目前Titan刚刚发布[Titan 0.5.0-M1](https://groups.google.com/forum/#!topic/aureliusgraphs/cNb4fKoe95M)版本，增加了很多特性，而且文档更加完善了，Titan 0.5.0 GA会在七月底发布，这会是一个非常接近1.0版本的产品，对于有需求的公司可以进行预研，对它完全掌握了再投入生产，毕竟Titan在实际生产环境的案例和技术文档都比较欠缺。但我相信Titan会成为下一代非常出色的Graph Database，我也会继续研究Titan和发布相关Titan相关的文章，希望能为Titan在中国推广做一些贡献，有感兴趣的同学欢迎一起讨论。




