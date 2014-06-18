---
layout: post
title: "Oracle DBA如何转型到Hadoop平台"
date: 2014-03-20 15:02:44 +0800
comments: true
categories: Hadoop
---
Oracle DBA如何转型到Hadoop平台，这篇博客是Cloudera的工程师Gwen Shapira的回答，因为他以前就是从事Oracle DBA的。原文在[这里](http://blog.cloudera.com/blog/2014/01/the-hadoop-faq-for-oracle-dbas/)。对于国内的一些DBA想转型Hadoop的可以参考一下。
<!--more--> 
>Q：**如果转型或者学习Hadoop平台的技术，Oracle DBA的经验还有用吗？**

A：在我合作的很多客户当中，我们和DBA团队一起合作将他们的数据仓库从Teradata或者Netezza迁移到Hadoop平台中，他们和我一起写Sqoop脚本、Oozie流程、Hive ETL脚本和Impala报表，几个月之后，我走了，原来的DBA团队已经掌握了Hadoop的技能了。  
我现在在Cloudera的解决方案架构师团队也会录用前DBA人员来作为解决方案顾问或系统工程师，我们认为他们的DBA经验非常有价值。
 


> Q：**公司会录用一个没有Hadoop工作经验的DBA吗，如果会，看重的是什么呢？**

A：我坚信DBAs完全有能力成为优秀的Hadoop专家 - 但这不代表所有的DBAs。下面是一些我觉得必备的技能：

- **适应命令行操作**：只会鼠标操作点击的DBAs和ETL工程师不合适
- **Linux经验**：Hadoop是运行在Linux上的，所以你需要非常适应Linux的文件系统、工具还有命令行，你还要理解CPU调度和IO等相关概念。
- **网络方面的知识**：[ISO7层网络架构](http://www.technology-training.co.uk/understandingtheiso7layermodel_10.php)，ssh原理，主机名称解析（/etc/hosts）,还有交换机。
- **良好SQL能力**：你需要熟练使用SQL语句，有数据仓库分区和并行处理经验、ETL经验和性能优化经验的优先。
- **编程能力**：不一定是Java，但是你要会写批处理脚本，不管是用Perl、Python还是其他语言，要会用伪代码来解决一些简单的问题，如果你不会编程，那就有问题了。
- **解决问题能力**：Hadoop没有Oracle那么成熟，所以你必须要学会用Google或者其他方式来搜索和解决问题，这个很重要。
- **更高级的职位，我还关注系统和架构方面的能力**：比如你做过飞行调度系统或者其他相似的东西。
- **因为我们需要面对客户，所以沟通能力也很重要**：如何倾听？如何向客户解释一个负责的技术问题？当我质疑你的时候该如何反应？

或许这些太多了，但是我觉得这些你转型Hadoop平台必备的技能。

> Q：**怎么开始学习Hadoop？**

A：首先你需要在自己的电脑上搭建一个Hadoop集群环境，这个网上的资料很多。然后你可以导入一些数据到Hadoop中来分析，这里有一个例子是用Flume导入Twitter数据然后用Hive来分析：

- http://blog.cloudera.com/blog/2012/09/analyzing-twitter-data-with-hadoop/
- http://blog.cloudera.com/blog/2012/10/analyzing-twitter-data-with-hadoop-part-2-gathering-data-with-flume/

还有就是买一些Hadoop方面的书籍，比如「Hadoop权威指南」、「Hadoop实战」等等。
推荐看下[这里](http://dongxicheng.org/mapreduce-nextgen/hadoope-every-day/)，是一些Hadoop方面的学习资源。

> Q：**我需要学习Java吗？**

A：你不需要成为一个特别专业的Java开发人员，但是你需要能够看懂Java源代码，因为Hadoop就是Java写的，还有Hive UDFS也需要Java来编写，相信我，不是很难，比SQL简单多了。

