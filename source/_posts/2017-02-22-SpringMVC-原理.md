---
title: 'SpringMVC: 原理'
date: 2017-02-22 13:16:18
categories:
- SpringMVC
---

# ApplicationContext在Web容器中的启动
* 应用程序级别的上下文（根上下文）：applicationContext.xml，一般加载非web的配置（Service、DataSource、DAO等）。
* web级别的上下文（子上下文）：dispatcher-servlet.xml，专注于mvc的配置（Controller、HandlerMapping、HandlerAdapter、ViewResolver等）。

web容器启动时首先建立根上下文，由`ContextLoaderListener`完成，子上下文是在`DispatcherServlet`初始化时建立。

# 请求分发和处理
`DispatcherServlet`是Spring MVC的核心组件，它负责接收HTTP请求并协调Spring MVC的各个组件完成请求处理的工作。
![](/images/springmvc/springmvc-framework.png)

1. 查找处理器，`DispatcherServlet`根据`HanlderMapping`查找`Handler`（xml配置、注解）进行查找。处理器映射器向前端控制器返回`HandlerExecutionChain`，处理器执行链封装了具体的`Handler`和拦截器链`HandlerInterceptor`。
2. 调用处理器适配器去执行`Handler`。
3. 执行`Handler`并返回`ModelAndView`。
4. 前端控制器请求视图解析器去进行视图解析，根据逻辑视图名解析成真正的视图。
5. 前端控制器进行视图渲染，将模型数据填充到Request。

## HandlerMapping
`HandlerMapping`在Spring启动时候就会注入到`DispatcherServlet`中，它的初始化方式主要依赖于AbstractHandlerMethodMapping这个抽象类，利用`initHandlerMethods(..)`方法获取所有Spring容器托管的bean，然后调用`isHandler(..)`看是否是`@Controller`注解修饰的bean，之后调用`detectHandlerMethods(..)`尝试去解析bean中的方法，也就是去搜索`@RequestMapping`注解修饰的方法，将前端请求的url path 和具体的 Method 来做关联映射。
完成所有的搜索bean搜索后，调用`registerHandlerMethod(..)`将 Method 构造为`HandlerMethod`，添加到HandlerMapping内含的属性列表中。`private final Map<T, HandlerMethod> handlerMethods = new LinkedHashMap<T, HandlerMethod>();`(T是泛型的，具体存放的最简单就是url path信息)。

## HandlerAdapter
调用`HandlerAdpter`时会遍历所有配置在Spring容器已经配好的 handlerAdapters，调用`supports(..)`方法，得到第一个返回为true的`HandlerAdapter`，传入参数实际上就是上面提到的就是`HandlerMethod`。 supports 主要就是验证某个`HandlerMethod`上定义的参数、返回值解析，是否能由该handlerAdapter处理。

## HandlerInterceptor
前端处理器在获取了`HandlerAdapter`后，就开始执行拦截器链，遍历依次调用`HandlerExecutionChain`的`applyPreHandle(HttpServletRequest request, HttpServletResponse response)`开始应用拦截器，一个一个调用其`preHandle()`方法；以及之后调用`applyPostHandle(HttpServletRequest request, HttpServletResponse response, ModelAndView mv)`并执行`postHandle()`方法。


* * *
感谢：
http://neoremind.com/2016/02/springmvc%E7%9A%84%E4%B8%80%E4%BA%9B%E5%B8%B8%E7%94%A8%E6%9C%80%E4%BD%B3%E5%AE%9E%E8%B7%B5/