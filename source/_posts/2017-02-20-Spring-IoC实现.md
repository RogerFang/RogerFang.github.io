---
title: 'Spring: IoC实现'
date: 2017-02-20 15:18:38
categories:
- Spring
tags:
- IoC
---



# IoC容器

Spring中有两个主要的容器系列：`BeanFactory`和`ApplicationContext`。

* `BeanFactory`: 定义了IoC容器的基本功能。

* `ApplicationContext`: 在简单容器的基础上，通过继承`MessageSource`、`ResourceLoader`、`ApplicationEventPublisher`等接口增加了许多面向框架的特性。



Spring通过定义`BeanDefinition`来管理各种Bean以及它们之间的依赖关系，是对它们的抽象，是IoC容器的核心数据结构。



## BeanFactory容器原理

在Spring中默认把`DefaultListableBeanFactory`作为一个功能完整的IoC容器。

![](/images/spring/spring-ioc-beanfactory.png)



```java 编程式使用IoC容器

ClassPathResource res = new ClassPathResource("spring-beans.xml");

DefaultListableBeanFactory factory = new DefaultListableBeanFactory();

XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(factory);

readr.loadBeanDefinitions(res);

```

Spring中使用`Resource`来封装`BeanDefinition`的信息源（XML文件），IoC容器通过`Resource`来定位`BeanDefinition`的信息，从而完成容器的初始化和依赖注入过程。



通过调用`XmlBeanDefinitionReader`的`loadDefinitions()`方法从`Resource`中开始载入`BeanDefinition`。



## ApplicationContext容器原理

`ApplicationContext`容器扩展了`MessageSource`接口，可以支持国际化的实现，开发多语言版本；具体的`ApplicationContext`实现都是继承了`DefaultResourceLoader`的子类，可以从不同的地方得到`BeanDefinition`的资源；同时还继承了`ApplicationEventPublisher`接口，从而在应用上下文中引入了事件机制，这些事件机制和Bean的生命周期的结合为Bean的管理提供了便利。



`ApplicationContext`是通过`refresh()`方法来启动IoC容器的，同时还会对`BeanDefinition`的信息源进行封装成`Resource`（例如 ClassPathResource、FileSystemResource等）。



# IoC容器的初始化

IoC容器的初始化分为3个过程：

1. `Resource` 的定位过程，即`BeanDefinition`的资源定位。

2. `BeanDefinition` 的载入和解析。

3. `BeanDefinition` 在IoC容器中的注册。



XML -> Resource -> Document -> BeanDefinition



## Resource定位

使用`DefaultListableBeanFactory`时，需要自己去定位`Resource`，并提供相应的`BeanDefinitionReader`来对这些信息进行处理。`DefaultListableFactory`是一个纯粹的IoC容器。



而在`ApplicationContext`中已经提供了一系列`Resource`定位的实现以及`BeanDefinitionReader`。



## BeanDefinition的载入和解析

IoC容器的载入过程，是把定义的Bean转化成Spring内部表示的数据结构（BeanDefinition）的过程。



`BeanDefinition`载入分为两部分，首先通过调用XML的解析器得到`Document`对象，然后通过`DocumentReader`完成对`Document`对象的解析（按照Spring的Bean规则，默认是使用`DefaultBeanDefinitionDocumentReader`）。



在`DefaultBeanDefinitionDocumentReader`中通过调用`BeanDefinitonParserDelegate`的`processBeanDefinition()`方法来实现，处理的结果是生成`BeanDefinitionHolder`对象持有。`BeanDefinitionHolder`对象除了持有`BeanDefinition`对象外，还有其他与`BeanDefinition`的使用相关的信息，比如 Bean 的名字、别名集合等。



## BeanDefinition在IoC容器中的注册

通过对`BeanDefinition`的载入和解析过程，在IoC容器内已经建立相应的数据结构和数据表示，但是这些数据还不能供IoC容器直接使用，需要对这些`BeanDefinition`对象进行注册。

IoC容器中通过一个`ConcurrentHashMap`来保持和维护这些加载的`BeanDefinition`。



注册完，整个IoC容器的初始化就完成了。



# IoC容器的依赖注入

> 依赖注入的过程是在用户第一次向IoC容器索要 Bean 时触发的，也可以通过设置`lazy-init=false`属性让容器完成对 Bean 的预实例化（其实也是一个完成依赖注入的过程）。



触发依赖注入的入口在`BeanFactory`提供的`getBean()`方法中。具体的依赖注入发生时在`BeanWrapper`的实现类`BeanWrapperImpl`中`setPropertyValue()`方法中实现。



在Bean的创建和对象依赖注入的过程中，需要根据`BeanDefinition`中的信息递归的完成。一个递归是在上下文中查找需要的Bean和创建Bean的递归调用。另一个递归是在依赖注入时，通过递归调用容器的`getBean()`方法，得到当前Bean依赖的Bean，同时也触发对依赖Bean的创建和注入。



# Bean的生命周期

1. Bean实例的创建

2. 为Bean实例设置属性

3. 调用Bean的初始化方法

4. 应用Bean

5. 容器关闭时调用Bean的销毁方法。