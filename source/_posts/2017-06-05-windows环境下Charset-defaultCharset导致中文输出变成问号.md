---
title: windows环境下Charset.defaultCharset导致中文输出变成问号
date: 2017-06-05 20:13:27
categories:
- 有趣的问题
tags:
- bug
---

**问题描述**
在使用 SpringMVC 进行前后台数据交互的时候，前台使用 Ajax 请求后台数据，后台读取数据库返回 Json，前台显示 Json 数据时中文都变成问号。


**解决过程**
一般中文编码问题都有几处源头，**数据库编码、传递json数据编码、前台页面编码**，这几部分不一致都可能导致中文乱码，可以肯定的是数据库编码和页面编码都是一致的`UTF-8`，但是查看项目环境中的 SpringMVC 的 `HttpMessageConverter` 等编码配置均是 `UTF-8`，这确实很奇怪，没有其他地方可以设置编码了，按道理说都是 `UTF-8` 是不会有问题的。

**问题所在**
某位前辈在项目代码中写了一个工具类对返回 Json 数据进行了封装，其中调用了`Charset.defaultCharset();`对 response headers中的`content-type`进行了设置。

Java doc中对`Charset.defaultCharset();`的定义是由当前启动的虚拟机以及本地环境的编码决定的。所以在windows环境下默认是`GBK`，故导致中文乱码。