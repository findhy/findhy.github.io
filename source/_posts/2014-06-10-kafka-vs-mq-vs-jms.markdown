---
layout: post
title: "Kafka vs MQ vs JMS"
date: 2014-06-10 10:43:08 +0800
comments: true
categories: Kafka
---
Kafka的介绍参加这篇文章：[Kafka Introduction](http://findhy.com/blog/2014/05/18/kafka/)，本文将Kafka、MQ、JMS这几个概念重新梳理一下，当然它们之间是有包含的关系，还有已经有了那么多的MQ产品，为什么LinkedIn还要开发Kafka呢？
<!--more-->

### 1.MQ ###
MQ的全称是Message Queue，中文名叫消息队列，是一种进程间通信或同一进程的不同线程间的通信方式，进程将消息存入链表数据结构中，然后其它拥有权限的进程去读取消息，这个过程是异步的，就是说消息接受者需要去轮询消息队列。详细可以参考Wikipedia的介绍：[Message Queue](http://zh.wikipedia.org/wiki/%E6%B6%88%E6%81%AF%E9%98%9F%E5%88%97)。

MQ是一种标准或者是一种通信方式，它是一个逻辑概念或者一个定义，不是一个具体的产品，但是有很多产品按照它定义的标准来实现，包括：JBoss Messaging、JORAM、**Apache ActiveMQ**、Sun Open Message Queue、Apache Qpid、HTTPSQS、Sparrow、Starling、**Kestrel**、**RabbitMQ**、Beanstalkd、**Amazon SQS**、**Kafka**、**ZeroMQ**、EagleMQ、IronMQ、FQueue、Spring AMQP、HornetQ、MQSSave、FFMQ、JMS4Spread、mom4j、OpenJMS、UberMQ等。这些都是Message Queue，关于这些产品的介绍可以去Google，或者参考下面两个链接：
http://stackoverflow.com/questions/731233/activemq-or-rabbitmq-or-zeromq-or  
http://www.open-open.com/53.htm。

### 2.JMS ###
JMS全称是Java Message Service，可见它是Java平台的Message Queue，而且它不是一个逻辑概念而是有一套面向消息中间件（MOM）的API，在javax.jms包内，但是要使用JMS，还要有一个JMS提供者来管理会话和队列，这个提供者就是JMS消息中间件（MOM），有哪些呢，开源的包括：Apache ActiveMQ、JBoss 社区所研发的 HornetQ、Joram、Coridan的MantaRay、The OpenJMS Group的OpenJMS等，商业的包括：BEA的BEA WebLogic Server JMS、TIBCO软件公司的EMS、GigaSpaces Technologies的GigaSpaces、IBM的WebSphere MQ等。JMS其它更多的可以参考Wikipedia的介绍：[JMS](http://zh.wikipedia.org/wiki/JMS)。

### 3.Kafka ###
Kafka的介绍以及特性还有它在hadoop生态系统中的位置，可以参考这篇文章：[Kafka Introduction](http://findhy.com/blog/2014/05/18/kafka/)，已经有了那么多的MQ产品，LinkedIn为什么还要再开发Kafka呢？目前业界比较成熟的MQ产品RabbitMQ、ZeroMQ、ActiveMQ，Kafka和它们相比有哪些优势？

关于RabbitMQ、ZeroMQ、ActiveMQ这几个的对比，可以参考这里：  
http://stackoverflow.com/questions/731233/activemq-or-rabbitmq-or-zeromq-or  
http://itindex.net/detail/43897-%E6%B6%88%E6%81%AF-%E4%B8%AD%E9%97%B4%E4%BB%B6-%E6%8A%80%E6%9C%AF  

Kafka与RabbitMQ比较可以参考这里：  
http://www.quora.com/RabbitMQ/RabbitMQ-vs-Kafka-which-one-for-durable-messaging-with-good-query-features

Kafka拥有更高的吞吐量和更好的分布式支持。






