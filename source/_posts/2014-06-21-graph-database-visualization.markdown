---
layout: post
title: "图数据库(graph database)可视化"
date: 2014-06-21 11:04:58 +0800
comments: true
categories: Titan
keywords: graph,database,visualization,图数据库(graph database)可视化
description: graph database visualization,图数据库(graph database)可视化
---
数据可视化是大数据的最后一环，重要性不可忽视，内行往往关心你的算法和架构，但是用户(客户或者领导)只会看最终展现在他们面前的东西，当然业务层面你需要先了解用户的核心需求，再去建模和设计指标，本文讨论关于数据可视化的技术方案，还有比较特别的图数据库的可视化。
<!--more-->
### 1.可视化类型 ###
从用户响应时间的角度，有：

- 静态可视化：当天日志导入到数据平台，晚上跑一些MR任务，再将执行结果存入HBase或者关系型数据库MySql，第二天运营看到结果
- 实时可视化：实时可视化前提是数据是实时变化的，通常用Storm或者Spark架构实时处理来自客户端或者Kafka的数据，再将处理完的数据导入到后台存储中，然后前端展现实时有两种方式，一种是被动的，就是你要再去刷新一次，这种比较好实现，一种是主动的，服务器端通知客户端，这种就用Websocket来做，如果服务器端用Node.js，更简单直接用socket.io

从展现的图类型来看，可能列举不全：

- 线性图：展现趋势
- 饼图/柱图：展现占比/对比
- 散点图：展现聚合关系
- 地理位置可视化：把数据在地图上展现，DataMap、GoogleMap都有接口，一般要求数据里面有地理位置属性（IP地址、城市、国家）
- Graph：用Graph展现一个KnowledgeMap，像社交网站的关系图谱，通常用来做用户定位和推荐

### 2.前端可视化技术 ###

基于浏览器的开源的：

- highchart
- d3.js
- ECharts
- google chart

基于桌面的开源的：

- [gephi](https://gephi.org/)

上面这些比较常用，而且例子和社区都比较成熟，具体的可以直接去官方看，我们现在普通报表有用highchart和d3.js的，gephi也可以在浏览器端展现。

### 3.Graph Database可视化 ###

关于Graph Database可视化单独拿出来，这块数据来源是像Titan、Neo4j和OrientDB这一类的图数据库，他们都提供基于REST的接口，以JSON数据返回Graph的信息，以Titan为例，在Titan上面可以搭建一个Rexster服务器，它提供很多针对Graph的REST接口，具体有哪些接口可以看这里[Rexster-REST-API](https://github.com/tinkerpop/rexster/wiki/Basic-REST-API)，前端通过调用接口获取数据，在前端去构建Graph图，构建的方式可以是Canvas或SVG，关于这两的比较可以看这里[SVG 与 Canvas：如何选择](http://msdn.microsoft.com/zh-cn/library/ie/gg193983\(v=vs.85\).aspx)。常用的技术解决方案有：

- [Sigma.js](http://sigmajs.org/)：开源，通用
- [Keylines](http://keylines.com/)：商业方案，官网有针对Titan和Neo4j可视化的例子
- [VivaGraph](https://github.com/anvaka/VivaGraphJS)：开源，通用，社区没有Sigma.js丰富
- [ngraph](https://github.com/anvaka/ngraph)：VivaGraph的下一个版本，使用WebGL支持3d效果展现
- [D3.js](http://d3js.org/)：开源，通用，上面提到了，它也提供Graph可视化的功能
- [Gephi](https://gephi.org/)：开源，通用，很强大的基于桌面可视化解决方案，通过插件也可以在浏览器端展现
- [Linkurious](http://linkurio.us/)：Neo4j专有的
- Neo4J web-admin：Neo4j专有的

参考：

http://stackoverflow.com/questions/14867132/is-d3-js-the-right-choice-for-real-time-visualization-of-neo4j-graph-db-data/23522907#23522907

http://stackoverflow.com/questions/18571685/neo4j-graph-visualizing-libraries?rq=1

