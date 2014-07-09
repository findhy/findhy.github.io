---
layout: post
title: "Spark1.0在YARN上部署过程"
date: 2014-06-28 14:13:34 +0800
comments: true
categories: Spark
keywords: Spark1.0在YARN上部署过程
description: Spark1.0在YARN上部署过程
---
Spark支持独立部署和在YARN上部署，选择在YARN上就是作为一个APP跑在YARN平台上，这样的好处是可以和原来的数据平台无缝结合。
<!--more-->
### 安装过程 ###
1、下载

    wget https://github.com/apache/spark/archive/v1.0.0.zip

2、解压

    unzip v1.0.0.zip

3、进入目录

    cd spark-1.0.0

4、设置maven内存参数：

    export MAVEN_OPTS="-Xmx2g -XX:MaxPermSize=512M -XX:ReservedCodeCacheSize=512m"

5、打包

    mvn -Pyarn-alpha -Dhadoop.version=2.0.0-cdh4.3.2 -DskipTests clean package

最后编译报错：  
    Failed to execute goal org.apache.maven.plugins:maven-antrun-plugin:1.7:run (default) on project spark-core_2.10: An Ant BuildException has occured: Please set the SCALA_HOME (or SCALA_LIBRARY_PATH if scala is on the path) environment variables and retry.
    [ERROR] around Ant part ...<fail message="Please set the SCALA_HOME (or SCALA_LIBRARY_PATH if scala is on the path) environment variables and retry.">... @ 6:126 in /home/hadoop/spark-cdh4.3.2/spark-1.0.0/core/target/antrun/build-main.xml

缺失scala的依赖  

查看pom.xml  
    <scala.version>2.10.4</scala.version>

6、安装scala2.10.4
    wget http://www.scala-lang.org/files/archive/scala-2.10.4.tgz
    
    tar -zxvf scala-2.10.4.tgz
    
    export SCALA_HOME=/opt/scala/scala-2.10.4

7、再次打包

    mvn -Pyarn-alpha -Dhadoop.version=2.0.0-cdh4.3.2 -DskipTests clean package

这个打包过程需要很久，看到BUILD SUCCESS就成功了

8、打spark内核包

    SPARK_HADOOP_VERSION=2.0.0-cdh4.3.2 SPARK_YARN=true sbt/sbt assembly

9、执行完成之后，在YARN上发布APP

    ./bin/spark-submit --class org.apache.spark.examples.SparkTC --master yarn-cluster --num-executors 3 --executor-memory 2g --driver-memory 4g --executor-cores 1 --jars /home/hadoop/spark-cdh4.3.2/spark-1.0.0/examples/target/spark-examples*.jar


看到命令行显示  
{% img /images/spark-1.png %}  

YARN管理台显示  
{% img /images/spark-2.png %}  

点击yarn app的tracking UI到达Spark的管理界面  
{% img /images/spark-3.png %}  


