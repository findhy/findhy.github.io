---
layout: post
title: "Caused by: java.io.NotSerializableException: kafka.javaapi.producer.Producer"
date: 2014-06-14 20:24:15 +0800
comments: true
categories: Kafka
---
我们现在的架构使用Kafka作为消息的入口，数据全部发送到Kafka中，然后用Storm的Topology写一个spout去订阅Kafka的消息，执行提交Topology：
<!--more-->
    storm jar storm-kafka-0.8-plus-test-0.1.0-SNAPSHOT-jar-with-dependencies.jar storm.kafka.topology.CyouStormTopology -c nimbus.host=10.0.1.254
但是报错：

    Exception in thread "main" java.lang.RuntimeException: java.io.NotSerializableException: kafka.javaapi.producer.Producer
	    at backtype.storm.utils.Utils.serialize(Utils.java:56)
	    at backtype.storm.topology.TopologyBuilder.createTopology(TopologyBuilder.java:89)
	    at storm.kafka.topology.CyouStormTopology.main(CyouStormTopology.java:32)
    Caused by: java.io.NotSerializableException: kafka.javaapi.producer.Producer
	    at java.io.ObjectOutputStream.writeObject0(ObjectOutputStream.java:1183)
	    at java.io.ObjectOutputStream.defaultWriteFields(ObjectOutputStream.java:1547)
	    at java.io.ObjectOutputStream.writeSerialData(ObjectOutputStream.java:1508)
	    at java.io.ObjectOutputStream.writeOrdinaryObject(ObjectOutputStream.java:1431)
	    at java.io.ObjectOutputStream.writeObject0(ObjectOutputStream.java:1177)
	    at java.io.ObjectOutputStream.writeObject(ObjectOutputStream.java:347)
	    at backtype.storm.utils.Utils.serialize(Utils.java:52)
	    ... 2 more
后来参考这里的提示：https://groups.google.com/forum/#!msg/storm-user/heQSeawhC5I/y40-FnQ4hiUJ  
修改代码如下：

    	@Override
    	public void prepare(Map stormConf, TopologyContext context,
    			OutputCollector collector) {
    		LOG.info("begin to prepare in bolt from CyouSendToKafkaBolt");
    		this._collerctor = collector;
    		
    		Properties props = new Properties();
    		props.put("metadata.broker.list", "master:9092");
    		props.put("serializer.class", "kafka.serializer.StringEncoder");
    		props.put("partitioner.class", "storm.kafka.producer.CyouPartitioner");
    		props.put("request.required.acks", "1");
    		ProducerConfig config = new ProducerConfig(props);
    
    		producer = new Producer<String, String>(config);
    	}
将原来在Bolt构造函数里面初始化Producer，改为在prepare(...)中去初始化，最后再次执行就没有问题了。

总结：在bolt中做一些初始化的代码，要放到prepare(...)方法中，而不要放到构造函数中，因为prepare(...)方法是在Storm worker JVM中被调用，而构造函数是在Nimbus JVM中被调用而造成不会被serialized。

