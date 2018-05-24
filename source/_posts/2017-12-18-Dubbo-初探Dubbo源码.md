---
title: 'Dubbo:初探Dubbo源码'
date: 2017-12-18 18:52:53
categories:
- dubbo
tags:
- dubbo
---

# dubbo简介
## 背景
![](/images/dubbo/dubbo-architecture-roadmap.png)

随着应用规模不断增大，应用之间的交互越多，基于MVC的垂直应用架构的开发模式弊端逐渐突显：开发维护成本变高，部署效率低等。
通过将核心业务抽取出来，作为独立的服务，逐渐形成稳定的服务中心，以实现服务的共享和重用，分布式服务框架成为了应用架构的核心。
流动计算架构可以解决当服务越来越多时，服务URL配置、服务路由、负载均衡、集群容错、容量规划等问题，资源调度和治理中心（SOA）是关键。

## dubbo架构
![](/images/dubbo/dubbo-architecture.png)

| 角色 | 说明 |
|--------|--------|
|Provider|服务提供方|
|Consumer|服务消费方，调用远程服务|
|Registry|服务注册与发现的注册中心|
|Monitor|服务监控中心，包括服务调用次数和调用时间等|
|Container|服务运行容器|

当服务集群规模进一步扩大，带动IT治理结构进一步升级，需要实现动态部署，进行流动计算，现有分布式服务架构不会带来阻力。下图是**未来可能的一种架构**：
![](/images/dubbo/dubbo-architecture-future.png)

| 角色 | 说明 |
|--------|--------|
|Deployer|自动部署服务的本地代理|
|Repository|仓库用于存储服务应用发布包|
|Scheduler|调度中心基于访问压力自动增减Provider|
|Admin|统一管理控制台|
|Registry|服务注册与发现的注册中心|
|Monitor|服务监控中心，包括服务调用次数和调用时间等|

# dubbo框架设计
![](/images/dubbo/dubbo-framework.png)

- 图中左边淡蓝背景的为服务消费方使用的接口，右边淡绿色背景的为服务提供方使用的接口，位于中轴线上的为双方都用到的接口。
- 图中从下至上分为十层，各层均为单向依赖，右边的黑色箭头代表层之间的依赖关系，每一层都可以剥离上层被复用，其中，Service 和 Config 层为 API，其它各层均为 SPI。
- 图中绿色小块的为扩展接口，蓝色小块为实现类，图中只显示用于关联各层的实现类。
- 图中**蓝色虚线**为初始化过程，即启动时组装链。**红色实线**为方法调用过程，即运行时调时链。紫色三角箭头为继承，可以把子类看作父类的同一个节点，线上的文字为调用的方法。

| 层 | 说明 |
|--------|--------|
|config配置层|对外配置接口，以`ServiceConfig`,`ReferenceConfig`为中心，可以直接初始化配置类，也可以通过 spring 解析配置生成配置类|
|proxy服务代理层|服务接口透明代理，生成服务的客户端 Stub 和服务器端 Skeleton, 以`ServiceProxy`为中心，扩展接口为`ProxyFactory`|
|registry注册中心层|封装服务地址的注册与发现，以服务 URL 为中心，扩展接口为`RegistryFactory`,`Registry`,`RegistryService`|
|cluster路由层|封装多个提供者的路由及负载均衡，并桥接注册中心，以 Invoker 为中心，扩展接口为`Cluster`,`Directory`,`Router`,`LoadBalance`|
|monitor监控层|RPC 调用次数和调用时间监控，以 Statistics 为中心，扩展接口为`MonitorFactory`,`Monitor`,`MonitorService`|
|protocol远程调用层|封将 RPC 调用，以 Invocation, Result 为中心，扩展接口为`Protocol`,`Invoker`,`Exporter`|
|exchange信息交换层|封装请求响应模式，同步转异步，以 Request, Response 为中心，扩展接口为`Exchanger`,`ExchangeChannel`,`ExchangeClient`,`ExchangeServer`|
|transport网络传输层|抽象 mina 和 netty 为统一接口，以 Message 为中心，扩展接口为`Channel`,`Transporter`,`Client`,`Server`,`Codec`|
|serialize数据序列化层|可复用的一些工具，扩展接口为`Serialization`,`ObjectInput`,`ObjectOutput`,`ThreadPool`|

## 调用链
![](/images/dubbo/dubbo-extension.png)

## 服务暴露时序图
![](/images/dubbo/dubbo-export.png)

## 服务引用时序图
![](/images/dubbo/dubbo-refer.png)


* * *
为了保证记录学习dubb源码的连贯性，该部分内容介绍源于dubbo官方文档。
[dubbo背景](http://dubbo.io/books/dubbo-user-book/)
[dubbo框架设计](http://dubbo.io/books/dubbo-dev-book/design.html)