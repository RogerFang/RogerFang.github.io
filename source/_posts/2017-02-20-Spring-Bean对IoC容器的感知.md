---
title: 'Spring: Bean对IoC容器的感知'
date: 2017-02-20 19:55:36
categories:
- Spring
---

如果Bean需要直接对IoC容器进行操作，就需要在Bean中设定对容器的感知。Spring IoC容器是通过特定的 aware 接口来完成的。

* `BeanNameAware`: 可以在 Bean 中得到它在IoC容器中的Bean 实例名称。
* `BeanFactoryAware`: 可以在 Bean 中得到Bean所在IoC容器，从而直接使用容器的服务。
* `ApplicationContextAware`: 可以在 Bean 中得到应用上下文。
* `MessageSourceAware`: 可以得到消息源。
* `ResourceLoaderAware`: 可以得到`ResourceLoader`，从而可以加载外部对应的`Resource`资源。

在设置Bean的属性之后，调用初始化回调方法（后置处理器`BeanPostProcessor`的两个方法）之前，Spring会调用aware接口中的setter方法。
