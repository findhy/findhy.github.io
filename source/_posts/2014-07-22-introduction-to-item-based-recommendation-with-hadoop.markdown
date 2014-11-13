---
layout: post
title: "Introduction to Item-Based Recommendations with Hadoop"
date: 2014-07-22 14:30:07 +0800
comments: true
categories: Mahout
keywords: Introduction to Item-Based Recommendations with Hadoop
description: Introduction to Item-Based Recommendations with Hadoop
---
本文是Mahout官网上的Item-Based CF on Hadoop的例子，原文在这里[Introduction to Item-Based Recommendations with Hadoop](https://mahout.apache.org/users/recommender/intro-itembased-hadoop.html)。
<!--more-->
### 前言 ###
Item-Based CF是协调过滤推荐算法的一个分支，是基于物品相似推荐，基于物品的 CF 的原理和基于用户的 CF 类似，只是在计算邻居时采用物品本身，而不是从用户的角度，即基于用户对物品的偏好找到相似的物品，然后根据用户的历史偏好，推荐相似的物品给他。从计算的角度看，就是将所有用户对某个物品的偏好作为一个向量来计算物品之间的相似度，得到物品的相似物品后，根据用户历史的偏好预测当前用户还没有表示偏好的 物品，计算得到一个排序的物品列表作为推荐。下图给出了一个例子，对于物品 A，根据所有用户的历史偏好，喜欢物品 A 的用户都喜欢物品 C，得出物品 A 和物品 C 比较相似，而用户 C 喜欢物品 A，那么可以推断出用户 C 可能也喜欢物品 C。    
{% img /images/mahout3.png %}  

### 示例 ###

#### 1、准备数据 ####
数据要求有三个字段：*userID, itemID and preference*，*userID*是用户ID，*itemID*是商品ID或者其它可推荐的物品ID，*preference*是用户对物品的偏好，可以是购买记录也可以是用户对物品的评分，如果*preference*没有值，Mahout会默认填1.0，比如我们用户购买商品的数据来进行推荐，这个偏好大家都是一样的，下面是准备的数据，没有填偏好字段：
	1,101
	1,102
	1,103
	2,101
	2,102
	2,103
	2,104
	3,101
	3,104
	3,105
	3,107
	4,101
	4,103
	4,104
	4,106
	5,101
	5,102
	5,103
	5,104
	5,105
	5,106
生成*input.txt*，将文件上传到HDFS中
    hadoop fs -put input.txt /user/hadoop/mahout/
#### 2、执行 ####
	mahout recommenditembased -s SIMILARITY_LOGLIKELIHOOD -i /user/hadoop/mahout/input.txt -o /user/hadoop/mahout/output --numRecommendations 25
#### 3、查看结果 ####
{% img /images/mahout2.png %} 

