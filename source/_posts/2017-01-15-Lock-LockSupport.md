---
title: 'Lock: LockSupport'
date: 2017-01-15 12:50:29
categories:
- Java锁
tags:
- 线程
- 锁
- Lock
- LockSupport
---

LockSupport工具类是用来创建锁Lock和其他同步类的基本线程阻塞原语。通过`park()`和`unpark()`方法来阻塞和唤醒线程。

LockSupport中的私有成员变量`UNSAFE`和`parkBlockerOffset`
```java
private static final sun.misc.Unsafe UNSAFE;
private static final long parkBlockerOffset;
```
**UNSAFE**：全名sun.misc.Unsafe用于可以直接操控内存，被JDK广泛用于自己的包中，如java.nio和java.util.concurrent。是JDK内部用的工具类。它通过暴露一些Java意义上说“不安全”的功能给Java层代码，来让JDK能够更多的使用Java代码来实现一些原本是平台相关的、需要使用native语言（例如C或C++）才可以实现的功能。该类不应该在JDK核心类库之外使用。

**parkBlockerOffset**：parkBlocker在内存中的偏移量，而parkBlocker是Thread类中的一个成员，用于记录线程被阻塞时是被谁阻塞的（可以用于线程监控和分析工具来定位原因）。
> 通过偏移量来获取和更改对象：这个parkBlocker就是在线程处于阻塞的情况下才会被赋值。线程都已经阻塞了，如果不通过这种内存的方法，而是直接调用线程内的方法，线程是不会回应调用的。


* * *
相关：
https://my.oschina.net/readjava/blog/282882