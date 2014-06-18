---
layout: post
title: "Storm-on-YARN Install"
date: 2014-05-17 06:37:52 +0800
comments: true
categories: Storm
---
Storm是一个流数据的实时计算框架，可以单独部署也可是部署在YARN上，本篇文章主要讲解Storm如何部署在YARN上面。  当然前提是Hadoop2.X已经装上了，然后需要安装Zookeeper来作为Storm集群的分布式协调服务。  
<!--more-->
环境说明  
    master 10.0.1.252  
    slave1 10.0.1.252  
    slave2 10.0.1.252  

软件版本  
    https://github.com/anfeng/storm-yarn/archive/master.zip  
    Storm-0.9.0-win21  
    zookeeper-3.4.5-cdh5.0.0-beta-2  
    apache-maven-3.1.0  

## Zookeeper集群搭建 ##
----------
Storm需要使用zookeeper来协调整个集群，但是storm并不用zookeeper来传递消息，所以zookeeper的负载很低，大多数情况下，单个节点的zookeeper就够了，所以我们这里就只部署一台机子的zookeeper，后面再扩展到集群。

### 1.安装JDK ###
前面Hadoop集群已经安装了，只要保证JDK版本在1.6以上就可以了。
### 2.下载和解压zookeeper安装包 ###
因为我们用的是CDH的Hadoop发行版，所以这里zookeeper也用CDH的，到这里下载：

    wget http://archive-primary.cloudera.com/cdh5/cdh/5/zookeeper-3.4.5-cdh5.0.0-beta-2.tar.gz
解压

    tar –zxvf zookeeper-3.4.5-cdh5.0.0-beta-2.tar.gz
### 3.修改zoo.cfg配置文件 ###
进入zookeeper安装目录/home/hadoop/zookeeper-3.4.5-cdh5.0.0-beta-2/conf，将zoo_sample.cfg重命名为zoo.cfg
    mv zoo_sample.cfg zoo.cfg

修改zoo.cfg文件，增加：
    tickTime=2000
    dataDir=/home/hadoop/zookeeper-data
    clientPort=2181
    initLimit=5
    syncLimit=2
    server.1=master:2888:3888

创建这个目录：/home/hadoop/zookeeper-data
### 4.修改myid ###
在/home/hadoop/zookeeper-data目录下创建文件myid，里面内容为server的ID，这里写：1

### 5.配置zookeeper的环境变量 ###
vi /etc/profile，增加：
    export ZOOKEEPER_HOME=/home/hadoop/zookeeper-3.4.5-cdh5.0.0-beta-2
    export PATH=$PATH:$ZOOKEEPER_HOME/bin

    source /etc/profile
### 6.启动zookeeper ###
    ./bin/zkServer.sh start
### 7.测试Zookeeper  ###
测试Zookeeper客户端是否可用

    ./bin/zkCli.sh -server 127.0.0.1:2181
测试结果，看到进入到了zookeeper的命令行就是成功了：

## Storm-on-YARN搭建 ##
----------

### 1.安装maven ###
下载maven安装包

    wget http://archive.apache.org/dist/maven/maven-3/3.1.0/binaries/apache-maven-3.1.0-bin.tar.gz

解压

    tar –zxvf apache-maven-3.1.0-bin.tar.gz
### 2.配置环境变量 ###
    vi /etc/profile
添加：
    export MAVEN_HOME=/home/hadoop/apache-maven-3.1.0
    export PATH=$PATH:$MAVEN_HOME/bin

使环境变量生效：
    source /etc/profile

测试  
    mvn –version
### 3.下载storm on yarn ###
这里下载@anfeng的分支版本（针对Hadoop2.2.0的），而不是官方版本。

    wget https://github.com/anfeng/storm-yarn/archive/master.zip
解压：

    unzip master
解压完了会生成一个storm-yarn-master目录
### 4.编译storm on yarn ###
进入storm-yarn-master目录：

    cd storm-yarn-master
编译（跳过测试）：  
    mvn package –DskipTests
### 5.在HDFS中创建对应Storm目录 ###
在HDFS中创建目录：

    /lib/storm/0.9.0-wip21
 
### 6.将storm.zip放进去（注意和你自己的目录对应） ###

    ./bin/hadoop fs -put /home/hadoop/storm-yarn-master/lib/storm.zip /lib/storm/0.9.0-wip21/
 
### 7.解压Storm ###
将/home/hadoop/storm-yarn-master/lib /storm-0.9.0-wip21.zip
解压到/home/hadoop/storm-yarn-master这个目录下
### 8.修改环境变量 ###
    export STORM_HOME=/home/hadoop/storm-yarn-master
    export PATH=$PATH:$STORM_HOME/storm-0.9.0-wip21/bin
    export PATH=$PATH:$STORM_HOME/bin
环境变量生效：

    source /etc/profile
### 9.修改storm.yaml ###
修改/home/hadoop/storm-yarn-master/storm-0.9.0-wip21/conf/storm.yaml配置文件，增加zookeeper的配置：
    storm.zookeeper.servers:
      - "master"
### 10.启动storm on yarn环境 ###
执行下面命令启动：

    storm-yarn launch storm.yaml

如果报错：yarn is not installed
设置Hadoop和YARN的环境变量（如果已经设置过了请跳过这一步）
    export HADOOP_DEV_HOME=/home/hadoop/hadoop-2.2.0-cdh5.0.0-beta-2
    export PATH=$PATH:$HADOOP_DEV_HOME/bin
    export PATH=$PATH:$HADOOP_DEV_HOME/sbin
    export HADOOP_MAPARED_HOME=${HADOOP_DEV_HOME}
    export HADOOP_COMMON_HOME=${HADOOP_DEV_HOME}
    export HADOOP_HDFS_HOME=${HADOOP_DEV_HOME}
    export YARN_HOME=${HADOOP_DEV_HOME}
    export HADOOP_CONF_DIR=${HADOOP_DEV_HOME}/etc/hadoop
    export HDFS_CONF_DIR=${HADOOP_DEV_HOME}/etc/hadoop
    export YARN_CONF_DIR=${HADOOP_DEV_HOME}/etc/Hadoop
    
    source /etc/profile

再试一下

    storm-yarn launch storm.yaml

 
因为storm是作为一个yarn程序运行在集群上的，所以在YARN的集群管理页面中会有一个AppId
 
### 11.复制storm.yaml ###
创建这个目录：~/.storm
执行（appid换成上面YARN管理界面中看到的appid）：

    storm-yarn getStormConfig -appId application_1399464020057_0006  -output ~/.storm/storm.yaml
### 12.查看nimbus ###
执行下面命令确定nimbus.host

    cat ~/.storm/storm.yaml | grep nimbus.host  


### 13.提交Topology ###
注意nimbus.host就是上面查询的nimbus服务器
    storm jar ./lib/storm-starter-0.0.1-SNAPSHOT.jar storm.starter.WordCountTopology WordCountTopology -c nimbus.host=10.0.1.253
 
### 14.Storm监控 ###
看一下storm的UI监控界面（nimbus.host:7070），可以看见刚刚提交的wordcountTopology
    http://nimbus.host:7070/
 
### 15.YARN-Storm Client ###
现在Storm作为一个应用部署在YARN上，后面所有Storm集群的控制都可以通过YARN客户端命令来完成，下面简单介绍一下。构建一个Storm的集群命令

    storm-yarn launch <storm-yarn-config>  
其中，<storm-yarn-config>是Storm的配置信息，包括启动Supervisor个数，Storm ApplicationMaster占用的内存等。

启动Storm后，可以通过下面的命令来控制Storm：

    storm-yarn [command] –appId [appId] –output [file] [–supervisors [n]]
其中，command为具体命令，具体参见下面两张表，参数 –appId 为启动的Storm在YARN中的应用程序ID，通过YANR的集群管理界面可以看见，-Supervisors为需要增加的Supervisor服务个数，该参数只对命令addSupervisors有效。
例如我们现在增加2个Supervisors，命令如下：

    storm-yarn addSupervisors -appId application_1399464020057_0006 -supervisors 2

输入storm-yarn查看所有命令

### 16.关闭Storm on yarn集群 ###

    storm-yarn shutdown –appId [applicationId]

## 参考 ##
http://dongxicheng.org/mapreduce-nextgen/storm-on-yarn/  

http://www.geedoo.info/storm-on-yarn-platform-to-build.html  

http://blog.csdn.net/weijonathan/article/details/17762477  

https://xumingming.sinaapp.com/179/twitter-storm-%E6%90%AD%E5%BB%BAstorm%E9%9B%86%E7%BE%A4/  

http://yzprofile.me/2013/04/25/storm-tutorial.html  


