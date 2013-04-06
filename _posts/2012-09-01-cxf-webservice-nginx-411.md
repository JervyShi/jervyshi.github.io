---
layout: post
title: "CXF webservice调用411 Length Required解决方案"
description: "项目中一直用到Apache CXF WebService，原本服务端都是apache做转发，一切都很正常。有一天服务端换成nginx做转发，问题就出来了，客户端调用时抛出 411 Length Required 异常。"
category: java
tags: [java, linux, webservice, nginx]
---
{% include JB/setup %}

项目中一直用到[Apache CXF WebService](http://cxf.apache.org/)，原本服务端都是apache做转发，一切都很正常。有一天服务端换成nginx做转发，问题就出来了，客户端调用时抛出 411 Length Required 异常。

	javax.xml.ws.soap.SOAPFaultException: Response was of unexpected text/html ContentType.  
	Incoming portion of HTML stream: <html>
	<head><title>411 Length Required</title></head>
	<body bgcolor="white">
	<center><h1>411 Length Required</h1></center>
	<hr><center>nginx</center>
	</body>
	</html>
		at org.apache.cxf.jaxws.JaxWsClientProxy.invoke(JaxWsClientProxy.java:146)
		at $Proxy53.testCXF(Unknown Source)

根据异常的提示大致分析出异常应该是在客户端向服务端发出请求时请求头中不包含”*Content-length*”，尝试用空对象调用接口异常并未抛出，只有当List中超过一定量如50的bean之后会抛出此异常。据此分析apache cxf对不同数据量进行请求时并不是采用同种方式，为了方便分析不同数据量请求头的区别，我找到一台Linux用nc命令监控8080端口，并将客户端数据发送至此Linux上的8080端口。

	# nc -l 8080

第一步:使用小数据量进行测试，获取请求头如下，发现可以调用成功时cxf发送数据时请求头中包含”*Content-Length*”

	POST / HTTP/1.1
	Content-Type: text/xml; charset=UTF-8
	SOAPAction: ""
	Accept: */*
	User-Agent: Apache CXF 2.2.4
	Cache-Control: no-cache
	Pragma: no-cache
	Host: 192.168.1.110:8080
	Connection: keep-alive
	Content-Length: 286

第二步:使用大数据量进行测试，获取请求头如下，发现可以调用失败时cxf发送数据时请求头中没有包含”*Content-Length*”，同时多了一个”*Transfer-Encoding*”值为”chunked”，关于 的介绍可以参考博文[http协议content-encoding & transfer-encoding](http://www.51testing.com/?uid-390472-action-viewspace-itemid-233985)

	POST / HTTP/1.1
	Content-Type: text/xml; charset=UTF-8
	SOAPAction: ""
	Accept: */*
	User-Agent: Apache CXF 2.2.4
	Cache-Control: no-cache
	Pragma: no-cache
	Host: 192.168.1.110:8080
	Connection: keep-alive
	Transfer-Encoding: chunked

通过一番搜索，发现apache默认就是支持*chunked*块接收的，而nginx如果要支持这种格式必须要添加*chunkin-nginx-module*模块，但是给nginx添加新模块需要重新编译nginx，虽然能解决现有问题但是有必要通过一系列的测试等，需要找到一个更简单的临时解决方案。

既然apache cxf可以在请求头中添加”*Content-Length*”来发送并且数据量达到一定值之后会转换为*chunked*块发送，那么是否可以关掉chunked块发送方式呢，感觉apache cxf官网完善的文档，经过一番探索在[Apahce cxf-User’s Guide](http://cxf.apache.org/docs/client-http-transport-including-ssl-support.html)中找到了”*AllowChunking*”的设置，有两种方式，可以直接在代码中设置*AllowChunking*也可以在配置文件中设置*AllowChunking*

	//Turn off chunking so that NTLM can occur
	Client client = ClientProxy.getClient(port);
	HTTPConduit http = (HTTPConduit) client.getConduit();
	HTTPClientPolicy httpClientPolicy = new HTTPClientPolicy();
	httpClientPolicy.setConnectionTimeout(36000);
	httpClientPolicy.setAllowChunking(false);
	http.setClient(httpClientPolicy);

也可以在配置文件中设置

	<http-conf:conduit name="{http://apache.org/hello_world_soap_http}SoapPort.http-conduit">
	  <http-conf:client Connection="Keep-Alive" MaxRetransmits="1" AllowChunking="false" />
	</http-conf:conduit>

配置文件中”*http-conf:conduit*”的name设置格式是”*{WSDL Namespace}portName.http-conduit*”，但是这种方式经常由于搞错WSDL Namespace与portName而导致设置不生效，其实name还有一种设置格式是路径匹配，假如我是服务端提供了一个服务，服务地址是”*http://jervyshi.tk/services/userService?wsdl*”，这个时候name只需要配置为”*http://jervyshi.tk/.\**”即可拦截”*http://jervyshi.tk*”下的所有服务。

	<http-conf:conduit name="http://jervyshi.tk/.*">
  		<http-conf:client AllowChunking="false" />
	</http-conf:conduit>

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>
