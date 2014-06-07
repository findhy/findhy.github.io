---
layout: post
title: "大数据 - 开放数据资源"
date: 2014-06-05 20:43:54 +0800
comments: true
categories: Freedata
---
大数据时代没有数据是玩不转的，真正拥有大数据的公司少之又少，更何况是个人开发者呢。数据有很多，最接近用户诉求、最能够精准定位用户需求的数据是最具有商业价值的，像搜索、电商、社交，而拥有这些数据的公司：Google、Facebook、百度、阿里巴巴、亚马逊等，它们首先会通过广告、增值服务等形式来将数据变现，肯定不会将数据开放出来，而且未来数据会越来越成为一个公司的核心竞争力。

对于大数据研究人员或者创业者，我们有哪些数据可以使用呢？我简单整理了一下。

### 1.数据分类 ###

我将数据分为两类：

- 静态大数据（Batch Data）：通常是一个备份数据集，容量往往很大
- 动态流数据（Streaming Data）：这是实时变动的数据，数据不断的更新和涌入
<!--more-->
### 2.数据来源 ###
#### 2.1 政府/非盈利机构（部分） ####
国内：  
http://www.stats.gov.cn/  
http://www.bosidata.com/  
http://www.cnnic.cn/  
http://www.eguan.cn/  


国外：  
如果你在AWS上，这上面的可以看看，直接从S3 get下来。  
http://aws.amazon.com/datasets  
 
freebase包含了很多开源项目的数据  
https://developers.google.com/freebase/data  

这是一个很有商业价值的数据，包含互联网上所有的网页信息，相当于Google的数据
http://commoncrawl.org/data/

想做一些项目，Wikipedia的数据足够了，它既有Batch Data，也有Streaming Data。
http://en.wikipedia.org/wiki/Wikipedia:Database_download  

这是一个开放数据源的集合，里面有很多公开的金融的社会统计数据，可以下载也可是可视化显示，非常适合研究和教学  
http://www.quandl.com/

#### 2.2 商业公司 ####

商业公司开放的数据通常都是有限制条件的，往往以一种合作的方式，而且基本上都是Streaming Data数据接口，案例有很多，像Twitter、腾讯、新浪微博，都有开放数据接口给个人开发者。  
Twitter：https://dev.twitter.com/docs/api/1.1/post/statuses/filter    

腾讯：http://open.qq.com/  

新浪微博：http://open.weibo.com/  

商业公司希望通过开放平台来构建一个以它为中心的生态系统，当然数据开放只是其中一部分。

#### 2.3 其它数据资源社区 ####
数据堂（主要是国内的政府机构数据）：http://www.datatang.com/


本文整理参考：    
http://www.quora.com/Where-can-I-find-large-datasets-open-to-the-public  

http://www.datapanda.net/123/  

http://www.zhihu.com/question/19969760  

