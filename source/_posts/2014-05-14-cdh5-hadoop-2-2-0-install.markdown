---
layout: post
title: "Hadoop 2.2.0-cdh5.0.0-beta-2安装说明"
date: 2014-05-14 23:08:37 +0800
comments: true
categories: Hadoop
---
环境说明  
    master 10.0.1.252  
    slave1 10.0.1.252  
    slave2 10.0.1.252  

软件版本  
    Hadoop 2.2.0-cdh5.0.0-beta-2  
    JDK 1.7.0_45

## 开始安装  ##

----------

###1.创建用户###
    useradd hadoop

###2.修改密码###
    passwd hadoop

###3.修改HOSTS文件###
    10.0.1.252 master  
    10.0.1.253 slave1  
    10.0.1.254 slave2  


###4.节点互信配置###
在每个节点执行  
    ssh-keygen -t dsa -P '' -f ~/.ssh/id_dsa
    cat ~/.ssh/id_dsa.pub >~/.ssh/authorized_keys

更改权限  
    [hadoop@master .ssh]$ chmod 600 authorized_keys
    [hadoop@master ~]$ chmod 700 .ssh/

在master节点操作  
拷贝其它节点的公钥到authorized_keys文件中  
    [hadoop@master .ssh]$ ssh hadoop@slave1 cat ~/.ssh/authorized_keys >> authorized_keys
    [hadoop@master .ssh]$ ssh hadoop@slave2 cat ~/.ssh/authorized_keys >> authorized_keys
然后将公钥拷贝到其它节点  
    [hadoop@master .ssh]$ scp authorized_keys hadoop@slave1:~/.ssh/
    [hadoop@master .ssh]$ scp authorized_keys hadoop@slave2:~/.ssh/

测试一下，不需要密码就可以访问    
    ssh master  
    Ssh slave1  
    Ssh slave2  
  
首次会让输入yes，后面就可以直接登录了

###5.JDK安装###

用java –version检查是否安装了JDK，如果没有安装，则参照下面的连接安装：
https://www.cloudera.com/content/cloudera-content/cloudera-docs/CDH5/latest/CDH5-Installation-Guide/cdh5ig_oracle_jdk_installation.html#topic_29_1

###6.载CDH安装包###
下载解压：http://archive-primary.cloudera.com/cdh5/cdh/5/hadoop-2.2.0-cdh5.0.0-beta-2.tar.gz
    tar –zxvf hadoop-2.2.0-cdh5.0.0-beta-2.tar.gz

###7.修改hadoop-env.sh###
修改${HADOOP_HOME}/etc/hadoop/hadoop-env.sh
    export JAVA_HOME=/usr/java/jdk1.7.0_45

###8.修改mapred-site.xml###
在${HADOOP_HOME}/etc/hadoop/目录中，将mapred-site.xml.templat重命名成mapred-site.xml： 
    mv mapred-site.xml.template mapred-site.xml
并添加以下内容：
    <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
    </property> 
###9.修改core-site.xml###
修改${HADOOP_HOME}/etc/hadoop/core-site.xml
    <property>
    <name>fs.default.name</name>
    <value>hdfs://master:8020</value>
    <final>true</final>
    </property>

###10.修改yarn-site.xml###
修改${HADOOP_HOME}/etc/hadoop/yarn-site.xml：
    <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
    </property>
    <property>
    <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
    <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>

###11.修改hdfs-site.xml###
修改${HADOOP_HOME}/etc/hadoop/hdfs-site.xml
    <property>
    <name>dfs.namenode.name.dir</name>
    <value>/home/hadoop/dfs/yarn/name</value>
    </property>
    <property>
    <name>dfs.datanode.data.dir</name>
    <value>/home/hadoop/dfs/yarn/data</value>
    </property>
    <property>
    <name>dfs.replication</name>
    <value>1</value>
    </property>
    <property>
    <name>dfs.permissions</name>
    <value>false</value>
    </property>
###12.修改slaves文件###
在slaves文件中添加你的slave节点：
    slave1
    slave2
###13.修改masters文件###
在masters文件中添加你的master节点：
    master
###14.将安装包复制到其它节点###
    scp -r /home/hadoop/hadoop-2.2.0-cdh5.0.0-beta-2 hadoop@slave1:/home/hadoop/hadoop-2.2.0-cdh5.0.0-beta-2
    scp -r /home/hadoop/hadoop-2.2.0-cdh5.0.0-beta-2 hadoop@slave2:/home/hadoop/hadoop-2.2.0-cdh5.0.0-beta-2

*上面配置文件对应的目录在每个节点都需要提前创建好*
###15.初始化HDFS###
    ./bin/hadoop namenode –format
###16.启动HDFS###
    ./sbin/start-dfs.sh
###17.启动YARN###
    ./sbin/start-yarn.sh
###18.测试集群###
自己创建一个文件put到HDFS的/test/in目录下面
执行：
    ./bin/hadoop jar share/hadoop/mapreduce2/hadoop-mapreduce-examples-2.2.0-cdh5.0.0-beta-2.jar wordcount /test/in/ /test/out/wordcountr


###19.YARN管理地址###
http://master:8089/cluster/cluster