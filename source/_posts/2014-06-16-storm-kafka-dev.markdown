---
layout: post
title: "Storm与Kafka集成开发"
date: 2014-06-16 16:44:35 +0800
comments: true
categories: Storm
---
项目源码在这里[storm-kafka](https://github.com/findhy/storm-kafka)，里面包括了简单说明和测试过程，照着做就可以了，下面简单介绍一下。

{% img /images/storm-kafka-dev1.png %} 
<!--more-->

- 数据源：该架构主要用来处理Streaming data，例子使用Wikipedia提供的Websocket接口，实时发送当前在Wikipedia网站编辑内容的用户相关信息
- Websocket：可以参考这篇文章[Java-Websocket](http://findhy.com/blog/2014/06/12/java-websocket/)，本框架使用Java语言作为kafka的客户端实现，所以也用的Java来实现Websocket，Kafka也支持其它语言作为client，这里是支持的客户端列表[kafka-client](https://cwiki.apache.org/confluence/display/KAFKA/Clients)，用Node.js也是一个不错的选择
- Storm：Storm与Kafka集成的重点在于Storm的Spout部分，这部分直接依赖这个库[storm-kafka-0.8-plus](https://github.com/wurstmeister/storm-kafka-0.8-plus)，实现订阅Kafka的Topic
- 数据出口：Storm对Streaming data处理完了之后，一般会有两种出口，一是将数据持久化到HBase/Cassandra/Redis这样的NoSql Database中，二是通知前端在可视化界面上实时变动，该框架实现的是Storm将处理完的数据再次发送到Kafka中，前端通过Node.js和socket.io去订阅这个数据就可以(这部分暂未实现)

