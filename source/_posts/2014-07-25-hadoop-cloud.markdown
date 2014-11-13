---
layout: post
title: "基于云的Hadoop架构"
date: 2014-07-25 12:18:55 +0800
comments: true
categories: AWS
keywords: 基于云的Hadoop架构
description: 基于云的Hadoop架构
---
今天看了Netflix分享的基于AWS的大数据平台Hadoop架构，感觉这是未来大数据平台的发展模式，将整个数据仓库部署在云端，开发者不需要考虑集群的资源扩展，也不需要接触复杂的底层命令，通过一个简单RESTFul接口就可以提交我们的MapReduce任务，还可以实时看到任务和运行状态。
<!--more-->
### Netflix基于AWS的Hadoop架构介绍 ###
{% img /images/hadoopcloud1.png %}  

Netflix的Hadoop云架构是完全基于AWS来搭建，底层存储采用S3，Hadoop组件采用AWS上原生的EMR，PAAS层是自己开发的Genie(目前已经开源)，开发者或者数据分析师通过Genie提供的RESTFul接口来提交和管理任务，由于AWS云带来的弹性扩展的好处，整个集群可以任意横向和纵向扩展，而开发者只需要专注在算法和业务上即可。

Netflix官方博客介绍：  
http://techblog.netflix.com/2013/01/hadoop-platform-as-service-in-cloud.html  

Infoq中文介绍：  
http://www.infoq.com/cn/news/2013/02/netflix-hadoop-PaaS  

Genie介绍PPT：  
{% slideshare 24063535 %}  

Genie已经开源：  
https://github.com/Netflix/genie  

Hadoop on OpenStack介绍PPT：   
{% slideshare 18948566 %}  


