---
layout: post
title: "Graph Database"
date: 2014-06-17 14:50:02 +0800
comments: true
categories: Titan
tags: [Titan,Graph database]
keywords: Titan, Graph database,Graph data structure
description: Titan,Graph database,Graph data structure
---
[Graph database](http://en.wikipedia.org/wiki/Graph_database)是近年兴起的NoSQL存储模型(graph, key-value, column, and document)中的一种实现数据库，代表产品是Neo4j、Titan，它的理论基础是[图论/Graph theory](http://zh.wikipedia.org/wiki/%E5%9B%BE%E8%AE%BA),一个数学分支，主要研究顶点和边组成的图形的数学理论和方法，它的起源也比较有意思，这部分本文不谈，感兴趣的同学可以去Google研究一下。
<!--more-->
想了解Graph Database，我们还是要把图论(Graph theory)简单说一下，略过中间复杂的数学问题，我们以结果为导向，图论可以解决哪些问题？这里有一个[图论经典问题简介](http://www.global-sci.org/mc/issues/3/no2/freepdf/67s.pdf),诸如[最短路径问题](http://zh.wikipedia.org/wiki/%E6%9C%80%E7%9F%AD%E8%B7%AF%E9%97%AE%E9%A2%98)、[旅行售货商问题](http://zh.wikipedia.org/wiki/%E6%97%85%E8%A1%8C%E6%8E%A8%E9%94%80%E5%91%98%E9%97%AE%E9%A2%98)、[中国邮递员问题](http://zh.wikipedia.org/wiki/%E9%82%AE%E9%80%92%E5%91%98%E9%97%AE%E9%A2%98)，这些是图论中研究的经典问题，具体实现会涉及到不同的算法问题。

举个例子，在社交网络中，你搜索了一个人，怎么最快结识他，这其实就是一个最短路径问题，LinkedIn已经有相关的推荐产品，可以去试一下。而且未来随着智能设备的普及个人数据会呈现爆炸性增长，就是人的属性和关联会更多更复杂，其实这个问题在阿里现在已经存在了，他收购了那么多产品，重要一点就是为了获取数据，现在阿里拥有一个人的网购数据(淘宝、天猫)、社交数据(陌陌、新浪微博)、浏览搜索数据(UC、优酷)等等，这么庞大的一个数据结构该怎么来描述，更关键的是怎么快速的去做定位和推荐，这也是Graph Database兴起的一个原因，未来Graph这块必然会大放异彩。

### 数据结构 ###

Graph Database的存储单元是：节点(nodes)、关系/边(edges)、属性(properties)  
{% img /images/GraphDatabase_1.png %}   

- 节点(nodes)：通常是一个实体，类似于社交网络中的人或者电商网络中的商品，节点不一定都是同一个类型的，比如我们构建一个Graph，节点是人和商品，之间的连接是谁买了哪个商品，这样我们可以很容易找到买相同的产品的人，去做关联推荐
- 属性(properties)：是与节点(nodes)相关的信息，通常是节点的描述，比如人的性别、年龄、地点、电话等信息
- 关系/边(edges)：用来连接节点(nodes)的，代表节点(nodes)与节点(nodes)之间具有某种关系，比如用户A和用户B是好友，用户A和用户C来自同一个地方，关系可以是有方向和无方向的，而且节点(nodes)之间可以有多条关系

Graph Database的数据结构使得它在处理数据关联问题更具优势，因为它的节点(nodes)原生的就是通过某种关系连接在一起，这样就减少了join这样耗费资源的操作。

### 算法 ###

Graph Database中很多问题都涉及到一些重要算法，下面列举一些，数据来源[Wikipedia-Graph-Algorithms](http://zh.wikipedia.org/wiki/%E6%9C%80%E7%9F%AD%E8%B7%AF%E9%97%AE%E9%A2%98)：

- 基本遍历：深度优先搜索、广度优先搜索、A*、Flood fill
- 最短路径：Dijkstra、Bellman-Ford、Floyd-Warshall 、Kneser图
- 最小生成树：Prim、Kruskal
- 强连通分量：Kosaraju算法、Gabow算法、Tarjan算法
- 图匹配：匈牙利算法、Hopcroft–Karp、Edmonds's matching
- 网络流：Ford-Fulkerson、Edmonds-Karp、Dinic 、Push-relabel maximum flow

### 常规操作 ###

类似于关系型数据库提供的CRUD操作，Graph Database也提供一系列指令来操作Graph，下面G代表一个Graph data structure，你可以把它想象为关系型数据库中的一张表。

- adjacent(G, x, y)：检查节点x和节点y之间是否有一个边(edge)
- neighbors(G, x)：列出所有与节点x有连接/边(edge)的节点
- add(G, x, y)：插入一个边(edge)，从x指向y
- delete(G, x, y)：删除一个从x指向y的边(edge)
- get_node_value(G, x)：返回节点x的相关属性
- set_node_value(G, x, a)：设置节点x的属性为a
- get_edge_value(G, x, y)：返回节点x和y之间的边(edge)的属性，边(edge)也是有属性的
- set_edge_value(G, x, y, v)：设置节点x和y之间的边(edge)的属性为v

