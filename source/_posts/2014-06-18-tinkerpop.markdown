---
layout: post
title: "TinkerPop"
date: 2014-06-18 15:09:59 +0800
comments: true
categories: Titan
keywords: Titan,TinkerPop,property graph model,Blueprints,Pipes,Gremlin,Rexster
description: Titan,什么是TinkerPop,什么是property graph model,什么是Blueprints，Blueprints与Titan的关系
---
[TinkerPop](http://www.tinkerpop.com/)是Graph领域的一系列开源工具包的集合。下面分别介绍：
<!--more-->
### Blueprints ###
上一篇文章我们讲[Graph Database理论知识](http://findhy.com/blog/2014/06/17/graph-database-data-structure/)的时候，提到了图论关于图(Graph)的定义：顶点和边组成的图形，也在后面提到了关于Graph的一系列操作，包括：插入顶点、插入边、获取路径等等。Blueprints是对图这种抽象模型的具体实现，官方定义：[Blueprints](https://github.com/tinkerpop/blueprints/wiki)是一系列[属性图模型接口(property graph model interface)](https://github.com/tinkerpop/gremlin/wiki/Defining-a-Property-Graph)，那么接下来，什么是属性图模型(property graph model)？满足下面三个条件的图(Graph)被称为属性图(property graphs)：

- 顶点(vertices)和边(edges)可以包含任意多的key/value的属性
- 方向性，边(edges)具有方向性，可以从一个顶点(vertices)指向另外一个顶点(vertices)
- 多样性，顶点(vertices)之间的关系边(edges)可以是不同的类型，就是说两个顶点(vertices)可以拥有多种不同类型的边(edges)

满足上述三个条件的graph被称为property graphs，下面展现一个property graphs的例子，数据格式可以是[GraphML](http://graphml.graphdrawing.org/index.html)或者[GraphSON](https://github.com/tinkerpop/blueprints/wiki/GraphSON-Reader-and-Writer-Library)，前者是[XML](https://github.com/tinkerpop/gremlin/blob/master/data/graph-example-1.xml)，后者[JSON](https://github.com/tinkerpop/gremlin/blob/master/data/graph-example-1.json)，当然JSON会更轻量级。

{% img /images/tinkpop-1.png %}

一个property graphs包含下面这些元素

- 一系列顶点(vertices)
	- 每一个顶点(vertex)有一个唯一标识
	- 每一个顶点(vertex)有一个或者多个指向其它顶点的边(edge)
	- 每一个顶点(vertex)有一个或者多个指向自己的边(edge)
	- 每一个顶点(vertex)包含了一个或多个由map定义的key/value属性
- 一系列边(edges)
	- 每一个边(edge)有一个唯一标识
	- 每一个边(edge)具有方向性指向一个顶点(vertex)
	- 每一个边(edge)有一个label来标识两个顶点(vertex)之间的关系
	- 每一个边(edge)包含了一个或多个由map定义的key/value属性

什么是property graphs搞明白之后，我们再来看Blueprints，Blueprints为属性图模型(property graph data model)提供了一套接口、实现还有测试用例，你可以把它想象成JDBC，JDBC对数据库的操作原语进行了封装和实现，只不过JDBC是用来操作关系型数据库，而Blueprints用来操作Graph Database。现在主流的Graph Database都支持Blueprints，而且在TinkerPop整个软件栈中，Blueprints是最底层的基础，就是其它的工具包都是基于它之上的封装和扩展。怎么使用Blueprints？

maven引入：  
    <dependency>
       <groupId>com.tinkerpop.blueprints</groupId>
       <artifactId>blueprints-core</artifactId>
       <version>2.5.0</version>
    </dependency>

样例代码：  
    Graph graph = new Neo4jGraph("/tmp/my_graph");
    Vertex a = graph.addVertex(null);
    Vertex b = graph.addVertex(null);
    a.setProperty("name","marko");
    b.setProperty("name","peter");
    Edge e = graph.addEdge(null, a, b, "knows");
    e.setProperty("since", 2006);
    graph.shutdown();


### Pipes ###

[Pipes](https://github.com/tinkerpop/pipes/wiki)是一个图数据处理的框架，可以将它理解为管道(Pipe),它最大的好处是管道(Pipe)的输出可以作为其它管道(Pipe)的输入，这样我们就可以实现类似于mapreducer的复杂运算。

### Gremlin ###

[Gremlin](https://github.com/tinkerpop/gremlin/wiki)是一个图遍历语言，可以用Gremlin来实现图的查询、分析和操作，Gremlin只能适用于支持Blueprints的图数据库，支持多种JVM语言：Java 和 Groovy，文档：[GremlinDocs](http://gremlindocs.com/)、[SQL2Gremlin](http://sql2gremlin.com/)。

### Frames ###
[Frames](https://github.com/tinkerpop/frames/wiki)是一个object-to-graph映射框架

### Furnace ###

[Furnace](https://github.com/tinkerpop/furnace/wiki)是一个Graph算法包

### Rexster ###

[Rexster](https://github.com/tinkerpop/rexster/wiki)是一个Graph Server

TinkerPop的维护人员来自不同的Graph Database产品厂商，像Neo4j、Titan、OrientDB、Bitsy，它在Graph Database领域的地位我理解就像JavaEE里面的Apache。在最新的[TinkerPop3.0](https://github.com/tinkerpop/tinkerpop3)版本的时候，TinkerPop将原本分散的各个工具包合并成了一个项目，并且增加了很多特性，Titan0.5版本还不支持TP3，将会在Titan1.0版本时支持，更多的可以看[TinkerPop3 Story/doc](http://www.tinkerpop.com/docs/current/)。

