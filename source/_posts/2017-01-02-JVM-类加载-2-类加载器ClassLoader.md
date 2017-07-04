---
title: 'JVM: 类加载(2)-类加载器ClassLoader'
date: 2017-01-02 23:10:37
categories:
- JVM
tags:
- JVM
- 类加载
- 类加载器
- ClassLoader
---

类加载器只用于完成整个类加载过程的加载（Loading）动作：类的加载阶段中“通过一个类的全限定名去获取描述此类的二进制字节流”这个动作放到Java虚拟机外部去实现，以便让程序代码自己决定如何去获取所需要的类。
> 类加载器在类层次划分、OSGI、热部署、代码加密等领域被广泛应用。

类在虚拟机中的**唯一性**由加载它的类加载器和这个类本身一同确立。因为每个类加载器都有独立的类名称空间。


# 双亲委派模型
分别从两种角度来划分类加载器的类型：
从Java虚拟机的角度来讲，只存在两种不同的类加载器：
* 启动类加载器（Bootstrap ClassLoader）：HotSpot中使用C\+\+实现，也有使用Java实现但关键方法仍然是使用JNI回调到C的实现上。（用户无法获取到Bootstrap ClassLoader实例）
* 其他的类加载器：这些都是由Java语言实现，独立于虚拟机外部，并且都继承自`java.lang.ClassLoader`。

从Java开发人员的角度来看，类加载器可以大致分为3种：
* 启动类加载器（Bootstrap ClassLoader）：这个类加载器负责将存放在<JAVA_HOME>/lib目录下的，或被`-Xbootclasspath`参数指定的路径中的，并且被虚拟机识别的，加载到虚拟机内存中。
* 扩展类加载器（Extension ClassLoader）：这个加载器由`sun.misc.Launcher$ExtClassLoader`实现，它负责加载<JAVA_HOME>/lib/ext或者被java.ext.dirs系统变量所指定的路径中的所有类库。（用户可以直接使用扩展类加载器）
* 应用程序类加载器（Application ClassLoader）：这个类加载器由`sun.misc.Launcher$AppClassLoader`实现。这个是由ClassLoader类中的`getSystemClassloader()`方法返回的，所以一般也称它为系统类加载器。它负责加载用户类路径（ClassPath）上所指定的类库，开发者可以直接使用这个类加载器，一般情况下这个就是程序中默认的类加载器。

> 类加载器之间的父子关系一般使用组合（Composition）关系来复用父加载器的。

![](/images/jvm/classloader-parents-delegate.png)
**类加载器的双亲委派模型的工作过程**：如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把请求委派给父类加载器去完成，每一个层次的类加载器都是如此，因此**所有的加载请求最终都应该传送到启动类加载器中**，只有当父类加载器反馈自己无法完成这个加载请求时，子加载器才会尝试自己去加载。

> 使用双亲委派模型来组织类加载器之间的关系，一个明显的好处就是：Java类随着它的类加载器一起具备了一种带有优先级的层次关系。

## 双亲委派模型的弊端
双亲委派模型使得检查类是否加载的委托过程是单向的，每个ClassLoader的职责非常明确，但是同时会带来一个问题，即**顶层的ClassLoader无法访问底层的ClassLoader所加载的类**。

Java 提供了很多服务提供者接口（Service Provider Interface，**SPI**），允许第三方为这些接口提供实现。常见的 SPI 有 JDBC、JCE、JNDI、JAXP 和 JBI 等，这些 SPI 的接口由 Java 核心库来提供，第三方的实现则由**应用类加载器**完成加载，所以会导致系统类无法访问应用类的问题。

比如，在系统类中，提供了一个接口，该接口需要在应用中得以实现，该接口还绑定了一个工厂方法，用于创建该接口的实例，而接口和工厂方法都在启动类加载器中。这时，就会出现该**工厂方法无法创建由应用类加载器加载的应用实例的问题**。
> **解决办法**：双亲委派模型的补充，通过**上下文加载器**（ContextClassLoader）完成。


# 上下文加载器
上下文加载器是在`Thread.getContextClassLoader()`中得到的，默认情况下，上下文加载器就是**应用类加载器**。这样就使得在创建类实例的时候可以交由**上下文加载器**去完成。

![](/images/jvm/context-classloader.png)

# 突破双亲委派模型
双亲模式的类加载方式是虚拟机默认的方式，但并非必须这么做，通过重载ClassLoader可以修改行为。比如**Tomcat**和**OSGI**框架，都有各自独特的类加载器顺序。

可以通过重载`loadClass()`、`findClass()`等方法来改变类加载次序。