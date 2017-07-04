---
title: 'JMM: volatile的内存语义'
date: 2017-01-07 14:05:43
categories:
- JMM
tags:
- JVM
- JMM
- volatile
---

volatile关键字可以保证变量的可见性，也就是新值能立即同步到主内存，以及使用前立即从主内存刷新。

**可见性**：对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写入。
**原子性**：对任意单个volatile变量的读/写具有原子性，但是对于复合操作如volatile++这种复合操作不具有原子性。

# volatile写-读内存语义
volatile**写**内存语义：当写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量值刷新到主内存。

volatile**读**内存语义：当读一个volatile变量时，JMM会把该线程对应的本地内存置为无效，然后从主内存中读取共享变量。

# volatile内存语义的实现
## 编译器重排序序规则
JMM针对**编译器**指定的volatile重排序规则表。
![](/images/jmm/volatile-reorder-rules.png)
从表中可以看出：
1. 当第二个操作是volatile写时，不管第一个操作是什么，都不能重排序。
	这个规则确保volatile写之前的操作不会被编译器重排序到volatile写之后。
2. 当第一个操作时volatile读时，不管第二个操作是什么，都不能重排序。
	这个规则确保volatile读之后的操作不会被编译器重排序到volatile读之前。
3. 当第一个操作时volatile写时，第二个操作是volatile读时，不能重排序。

## 处理器内存屏障
为了实现volatile内存语义，编译器在生成字节码时，会在指令序列中插入**内存屏障**来禁止特定类型的**处理器**重排序。

# 增强的volatile内存语义
JSR-133之前的JMM虽然不允许volatile变量之间的重排序，但是**允许volatile变量和普通变量重排序**。
![](/images/jmm/advanced-volatile-memory-semantic.png)
在旧的JMM中，当1和2之间没有数据依赖关系时，1和2之间就可能被重排序（3和4类似），这样就会导致：读线程B执行4时，不一定能看到写线程A在执行1时对共享变量的修改。

也就是说，在旧的Java内存模型中，volatile的写-读没有**锁**的释放-获取所具有的的内存语义，**为了提供一种比锁更轻量级的线程通信机制**，JSR-133决定增强volatile的内存语义：
> 严格限制编译器和处理器对**volatile变量**与**普通变量**的重排序，*确保volatile的写-读和锁的释放-获取具有相同的内存语义*。