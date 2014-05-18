---
layout: post
title: "Protocol Buffers vs Avro vs Thrift"
date: 2014-05-18 15:53:37 +0800
comments: true
categories: RPC
---
分布式架构一个重要的思路是解耦，将系统拆解为很多个相互独立的组件，每个组件通过接口对外提供服务，这种面向服务（SOA）架构设计可伸缩性更强、维护成本也更低。但服务的管理、分布式锁、高效的组件调用等相关技术复杂性也更高。本文介绍几个常用的高效的RPC框架。

### 1.RPC模型 ###

一个简单的RPC调用模型如下图所示：  
{% img /images/rpc_1.png %} 

当我们在选型RPC框架的时候需要关注的几个核心问题是：

- 传输的协议和数据（JSON、XML等）是什么？
- 如何高效的数据存储和传输？
- 服务器端处理请求的方式？

### 2.不建议的方案 ###
- SOAP：基于XML，传输的数据太多
- CORBA：多度设计而且重量级，http://en.wikipedia.org/wiki/Common_Object_Request_Broker_Architecture
- DCOM, COM+ :主要用于windows客户端程序
- HTTP/JSON/XML/Plain Text：基于HTTP协议的，这种在简单的场景下是可以用的，像hessian，但缺点是缺乏更复杂的协议描述，只能传输简单的对象，而且JSON/XML都太重了

### 3.建议方案 ###
- Protocol Buffers
- Apache Thrift
- Apache Avro
- Message Pack
- kryo
- BSON

上面这些序列化框架共同的特点的是：有接口描述（IDL）、性能较高、版本控制和基于二进制的数据传输。下面重点介绍前三个。

<!--more-->

### 4.Protocol Buffers ###
- 来源Google，于2001年开始设计，2008年开源
- 非常稳定，大量运用在Google的生产环境中
- 目前支持四种语言：C++, Java, Python, and JavaScript 
- https://code.google.com/p/protobuf/

### 5.Apache Thrift ###
- 由Google-X实验室的工程师于2007年设计，后来主要用于Facebook内部项目，现在为apache的开源项目
- 旨在成为下一代的PB（更加全面的功能和支持更多的语言）
- 支持更多语言，包括：C++, Java, Python, PHP, Ruby, Erlang, Perl, Haskell, C#, 
Cocoa, JavaScript, Node.js, Smalltalk, OCaml and Delphi等
- IDL描述语言要比PB更简洁
- 提供一个RPC调用栈
- http://thrift.apache.org/

### 6.Apache Avro ###
- Avro是Hadoop的作者Doug Cutting写的另外一个RPC框架
- 支持序列化为JSON或者二进制
- 支持从protobufs和thrift读写数据
- 支持语言：Java, C, C++, C#, Python, Ruby 
- http://avro.apache.org/


### 7.传输数据大小比较 ###
{% img /images/rpc_2.png %} 

Thrift-TBinaryProtocol - 没有经过压缩的二进制协议，比文本协议要快，但是很难调试  
Thrift-TCompactProtocol - 经过压缩处理的，更加高效

### 8.哪些项目在使用Thrift？ ###
- Facebook 
- Cassandra project 
- Hadoop supports access to its HDFS API through Thrift bindings 
- HBase leverages Thrift for a cross-language API 
- Hypertable leverages Thrift for a cross-language API since v0.9.1.0a 
- LastFM 
- DoAT 
- ThriftDB 
- Scribe 
- Evernote uses Thrift for its public API. 
- Junkdepot

### 9.哪些项目在使用Protocol Buffers？ ###
- Google 
- ActiveMQ uses the protobuf for Message store 
- Netty (protobuf-rpc) 

### 10.不同框架测试结果比较 ###
https://code.google.com/p/thrift-protobuf-compare/wiki/BenchmarkingV2

### 11.总结 ###
- 其实大部分场景下，基于HTTP的Hessian或者阿里巴巴的Dubbo完全可以满足需求
- 如果追求更高的性能，像高并发的游戏服务器端，可以选择Protocol Buffers、Avro或Thrift
- Protocol Buffers和Thrift的稳定性要更好，大量运用于Google和Facebook的生产环境中
- Thrift相比Protocol Buffers支持更多的语言，框架逻辑更加清晰，易于定制扩展，性能与Protocol Buffers不相上下，目前来说应该是最佳的选择
- Avro的优势在于Schema动态加载功能，而且Avro更适合于数据交换及存储的通用工具和平台
- Avro的性能不输Protocol Buffers和Thrift，但缺点是发展时间较短，没有经过太多项目的验证


### 12.参考 ###
http://www.slideshare.net/ChicagoHUG/avro-chug-20120416 
http://www.slideshare.net/IgorAnishchenko/pb-vs-thrift-vs-avro 
http://ganges.usc.edu/pgroupW/images/a/a9/Serializarion_Framework.pdf  
https://www.igvita.com/2011/08/01/protocol-buffers-avro-thrift-messagepack/http://www-old.itm.uni-luebeck.de/teaching/ws1112/vs/Uebung/GrossUebungNetty/VS-WS1112-xx-Zero-Copy_Event-Driven_Servers_with_Netty.pdf?lang=de    
http://www.alidata.org/archives/1307  

