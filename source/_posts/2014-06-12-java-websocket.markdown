---
layout: post
title: "Java-WebSocket"
date: 2014-06-12 22:51:51 +0800
comments: true
categories: WebSocket
---
本文涉及到几个概念后面细说，Websocket协议、socket.io、Java-WebSocket，核心都是围绕Websocket，最后重点讲解在Java端实现Websocket的访问。
<!--more-->

### 1.Websocket协议 ###
首先解释一下Websocket协议，[Wikipedia](http://zh.wikipedia.org/wiki/WebSocket)的定义是：
> WebSocket是HTML5开始提供的一种浏览器与服务器间进行全双工通讯的网络技术

什么是【全双工】？可以理解为双方经过对方确认可以互传数据了。客户端和服务器端建立[TCP](http://zh.wikipedia.org/wiki/%E4%BC%A0%E8%BE%93%E6%8E%A7%E5%88%B6%E5%8D%8F%E8%AE%AE)连接的时候需要经过三次握手过程才能达到全双工的连接状态。在WebSocket API中，浏览器和服务器只需要做一个握手的动作，然后，浏览器和服务器之间就形成了一条快速通道。两者之间就直接可以数据互相传送。

在Websocket协议之前，浏览器和服务器端最常用的是HTTP协议，而HTTP协议是半双工，服务器端无法实时向客户端发送消息，但是很多场景下我们又有这样的需求，以前的解决方案是浏览器端做轮询或者用Comet做长连接，两种方案都比较消耗资源。

Websocket协议的出现完全解决了这个问题，服务器端可实现主动推送数据到客户端。现在主流的浏览器Chrome、Firefox等都已经支持Websocket协议了。

而在服务器端，主流的web服务器和编程语言都对Websocket进行了支持，下面摘自Wikipedia：

- php - http://code.google.com/p/phpwebsocket/
- jetty - http://jetty.codehaus.org/jetty/ (版本7开始支持websocket)
- netty - http://www.jboss.org/netty
- ruby - http://github.com/gimite/web-socket-ruby
- Kaazing - http://www.kaazing.org/confluence/display/KAAZING/Home
- Tomcat - http://tomcat.apache.org/ (7.0.26支持websocket)
- WebLogic - http://www.oracle.com/us/products/middleware/cloud-app-foundation/- weblogic/overview/index.html (12.1.2 开始支持)
- node.js - https://github.com/Worlize/WebSocket-Node
- node.js - http://socket.io
- nginx - http://nginx.com/
- mojolicious - http://mojolicio.us/
- python - https://github.com/abourget/gevent-socketio
- Django - https://github.com/stephenmcd/django-socketio
- Java - http://java-websocket.org/

### 2.socket.io ###
[socket.io](http://socket.io/)一个是基于Node.js架构体系的，支持Websocket的协议用于时时通信的一个软件包。socket.io给跨浏览器构建实时应用提供了完整的封装，socket.io完全由javascript实现。Node.js实现Websocket访问的方式有很多种，socket.io是最常用的的。

和Node.js一样，socket.io是基于事件驱动模型，它使用Websocket协议，但同时对Adobe Flash sockets、JSONP polling、AJAX long polling也提供支持，它的源代码在这里[socket.io-github](https://github.com/Automattic/socket.io)。

所以socket.io是一个对Websocket协议封装的javascript库，我们可以用它来完成客户端和服务器端的代码。

### 3.Java-WebSocket ###
我们有时会有这样的需求，就是在后台Java端去调用一个Websocket接口的服务，比如在Android端去请求一个公共Websocket服务，[Java-WebSocket](http://java-websocket.org/)就是为了这样的需求而产生的，你可以完全用Java代码来实现Websocket的服务器端和客户端，而且它支持WSS，WSS是安全的Websocket连接，类似于HTTPS，下面我们来看一个客户端的例子，用Java实现用读取Wikipedia提供的Websocket接口：当前谁在Wikipedia上修改了文章。

POM文件添加依赖  
    <dependency>
		<groupId>org.java-websocket</groupId>
		<artifactId>Java-WebSocket</artifactId>
		<version>1.3.0</version>
	</dependency> 

客户端Java类  
    package storm.kafka.websocket;
    
    import java.net.URI;
    import java.net.URISyntaxException;
    
    import org.java_websocket.client.WebSocketClient;
    import org.java_websocket.drafts.Draft;
    import org.java_websocket.drafts.Draft_10;
    import org.java_websocket.framing.Framedata;
    import org.java_websocket.handshake.ServerHandshake;
    
    /**
     * This example demonstrates how to create a websocket connection to a server.
     * Only the most important callbacks are overloaded.
     */
    public class TestWebsocket extends WebSocketClient {
    
    	public TestWebsocket(URI serverUri, Draft draft) {
    		super(serverUri, draft);
    	}
    
    	public TestWebsocket(URI serverURI) {
    		super(serverURI);
    	}
    
    	@Override
    	public void onOpen(ServerHandshake handshakedata) {
    		System.out.println("opened connection");
    		// if you plan to refuse connection based on ip or httpfields overload:
    		// onWebsocketHandshakeReceivedAsClient
    	}
    
    	@Override
    	public void onMessage(String message) {
    		System.out.println("received: " + message);
    	}
    
    	public void onFragment(Framedata fragment) {
    		System.out.println("received fragment: "
    				+ new String(fragment.getPayloadData().array()));
    	}
    
    	@Override
    	public void onClose(int code, String reason, boolean remote) {
    		// The codecodes are documented in class
    		// org.java_websocket.framing.CloseFrame
    		System.out.println("Connection closed by "
    				+ (remote ? "remote peer" : "us"));
    	}
    
    	@Override
    	public void onError(Exception ex) {
    		ex.printStackTrace();
    		// if the error is fatal then onClose will be called additionally
    	}
    
    	public static void main(String[] args) throws URISyntaxException {
    		TestWebsocket c = new TestWebsocket(new URI("ws://wikimon.hatnote.com:9000"),
    				new Draft_10()); // more about drafts here:
    									// http://github.com/TooTallNate/Java-WebSocket/wiki/Drafts
    		c.connect();
    	}
    
    }
    
执行结果如下，可以接受到Wikipedia返回的当前修改的具体信息：  
    opened connection
    received: {"action": "edit", "change_size": 7, "flags": "M", "is_anon": false, "is_bot": false, "is_minor": true, "is_new": false, "is_unpatrolled": false, "ns": "Main", "page_title": "Siege of Amida (502\u2013503)", "parent_rev_id": "612641484", "rev_id": "575212259", "summary": "categorized for missing coordinate data", "url": "http://en.wikipedia.org/w/index.php?diff=612641484&oldid=575212259", "user": "Kevinsam"}
    received: {"action": "edit", "change_size": 38, "flags": null, "geo_ip": {"areacode": "", "city": "", "country_code": "US", "country_name": "United States", "ip": "71.167.104.60", "latitude": 38, "longitude": -97, "metro_code": "", "region_code": "", "region_name": "", "zipcode": ""}, "is_anon": true, "is_bot": false, "is_minor": false, "is_new": false, "is_unpatrolled": false, "ns": "Main", "page_title": "KMOV", "parent_rev_id": "612641485", "rev_id": "612641166", "summary": "/* History */", "url": "http://en.wikipedia.org/w/index.php?diff=612641485&oldid=612641166", "user": "71.167.104.60"}

### 4.总结 ###
Websocket协议是为了解决服务器推送数据到客户端而出现的由于HTTP协议的解决方案，基于Websocket协议又有很多实现，实现包括客户端和服务器端，客户端体现在主流的浏览器现在基本上都支持Websocket协议，服务器端体现在各个web服务器都对Websocket提供支持，各个编程语言也提供相应的实现包，让我们针对Websocket的开发更容易。

