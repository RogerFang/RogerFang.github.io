---
title: 'Tomcat(6): session实现'
date: 2017-03-05 22:27:31
categories:
- tomcat
tags:
- session
---

Http是无状态协议，解决办法：
* **cookie**
	cookie 技术是客户端的解决方案，cookie 就是由服务器发给客户端的特殊信息，而这些信息以文本文件的方式存放在客户端，然后客户端每次向服务器发送请求的时候都会带上这些特殊的信息。
* **session**
	session技术则是服务端的解决方案，它是通过服务器来保持状态的。在创建了 Session 的同时，服务器会为该 Session 生成唯一的 Session id，而这个 Session id 在随后的请求中会被用来重新获得已经创建的 Session。（Session id需要借助cookie存放在客户端，或者通过url重写实现）

# session实现
tomcat的session实现机制在`org.apache.catalina.session`包中，tomcat 对外提供 session 调用的接口在`javax.servlet.http`包中的`HttpSession`。实现包中的`StandardSession`是 tomcat 提供的标准实现（**用户不能通过 StandardSession 直接操作 Session**），而 servlet 操作 session 是通过`StandardSessionFacade`进行的（门面模式，这样可以防止程序员直接操作 StandardSession 所带来的安全问题）。

实现包中有用来管理 session 的工具类，负责创建和销毁 session 对象，其中`ManagerBase`是 session 管理工具类的基类，具体实现 session 管理功能的类都要继承这个类。`StandardManager`是 tomcat 默认的管理实现类，它会将 session 的信息存储到 web 容器所在服务器的内存中。`PersistentManagerBase`继承`ManagerBase`类，是所有持久化存储 session 信息的基类。tomcat还提供了`StoreBase`的抽象类用于持久化存储 session，tomcat 还给出了文件存储 `FileStore`和数据库存储`JDBCStore`两个实现。

**Session Id** 的生成机制是：一个随机数 + 时间 + JVM的id值。
JVM的id值是根据服务器硬件信息计算得来的，因此不同JVM的id值都是唯一的。

# session带来的问题
## 性能问题
**问题**：session实现机制弥补了 http 协议的无状态的特点，服务器端会占用一定的内存和CPU用来存储和处理session计算的开销。

**解决办法**：在web容器tomcat之前加一个静态资源服务器，apache或nginx，静态资源服务器没有解决 http 无状态问题的功能。

## session同步问题
**问题**：当服务器的数量大于或等于两台时，这就导致一个web容器里生成的 session id通过请求到了另一个web容器无法被识别。

**解决办法**
1. apache + tomcat + mod_jk
	这种解决方案，当一个web容器里session信息发生变化后，该web容器会向另一个web容器进行广播，另一个web容器收到广播后将session信息同步到自己的容器里。
    但是这个过程十分消耗系统资源，当访问量增加会严重影响到网站的效率和稳定性。
2. session存储独立（分布式session）
	将session信息存储独立于web容器及其所在服务器。
