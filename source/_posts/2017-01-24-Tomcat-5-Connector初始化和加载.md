---
title: 'Tomcat(5): Connector初始化和加载'
date: 2017-01-24 22:01:16
categories:
- tomcat
---


Connector 组件的主要任务：
1. 监听端口，创建服务端与客户端的连接；
2. 获取客户端请求的Socket数据，并对Socket数据进行解析和包装成Http请求数据格式；
3. 将包装后的数据交给Container处理。

Connector有两个属性：`protocolHandler`（协议处理器）和`adapter`（适配器），其中`protocolHandler`完成步骤(1)(2)，`adapter`完成步骤(3)。


三种不带Ajp的协议，客户端与Tomcat服务器**直接连接**：
* Http11NioProtocol----------默认方式，NIO方式的支持http1.1协议
* Http11Nio2Protocol----------NIO2方式的支持http1.1协议
* Http11AprProtocol----------使用ARP技术的支持http1.1协议（ARP：Apache portable runtime)

三种带Ajp的协议为定向包协议，即WEB服务器通过 TCP连接和SERVLET容器连接，例如tomcat和Apache、Nginx等前端服务器连接：
* AjpNioProtocol---------------NIO方式的支持Ajp协议
* AjpNio2Protocol---------------NIO2方式的支持Ajp协议
* AjpAprProtocol---------------使用ARP技术的支持Ajp协议

# Connector配置
```xml conf/server.xml
<Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000" redirectPort="8443" />
<Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />
```

# Connector初始化


# Connector启动

* * *
感谢：
阿里技术专家，楚岩：[Tomcat源码分析系列博客](https://yq.aliyun.com/tags/type_blog-tagid_2702/?spm=5176.8091938.0.0.FiZvcG)