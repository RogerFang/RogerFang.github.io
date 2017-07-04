---
title: 'Lock: 并发工具类-CountDownLatch'
date: 2017-01-15 14:20:41
categories:
- Java锁
tags:
- 线程
- 锁
- Lock
- CountDownLatch
---

在JDK并发包提供了几个常用的并发工具类：CountDownLatch、CyclicBarrier、Semaphore提供了一种**并发流程控制**的手段，而Exchanger工具类提供了在**线程间交换数据**的一种手段。

**CountDownLatch允许一个或多个线程等待其他线程完成操作。**

# CountDownLatch和CyclicBarrier的区别
1. CountDownLatch的作用是允许1或N个线程等待其他线程完成执行；而CyclicBarrier则是允许一组线程相互等待。
2. CountDownLatch的计数器无法被重置，只能使用一次；CyclicBarrier的计数器可以被重置后使用，因此它被称为是循环的barrier，能处理更为复杂的业务场景。
3. CountDownLatch通过自定义同步器（共享锁）实现的；而CyclicBarrier是通过重入锁ReentrantLock（排他锁）和Condition来实现的。


# 源码分析
## 依赖
CountDownLatch是通过自定义同步器以实现**共享锁**来完成`await()`和`countDown()`操作的。

CountDownLatch的await方法可以在多个线程中调用，当CountDownLatch的计数器为0后，调用await的方法都会依次返回。 也就是说可以多个线程同时在等待await方法返回，所以获取的是一个共享锁。

```java
private static final class Sync extends AbstractQueuedSynchronizer {
    private static final long serialVersionUID = 4982264981922014374L;

    Sync(int count) {
        setState(count);
    }

    int getCount() {
        return getState();
    }

	/**
     * 尝试获取共享锁。
     * 如果"锁计数器=0"，即锁是可获取状态，则返回1；否则，锁是不可获取状态，则返回-1。
     */
    protected int tryAcquireShared(int acquires) {
        return (getState() == 0) ? 1 : -1;
    }

	/**
     * 释放共享锁，将“锁计数器”的值减1
     */
    protected boolean tryReleaseShared(int releases) {
        // Decrement count; signal when transition to zero
        for (;;) {
            int c = getState();
            if (c == 0)
                return false;
            int nextc = c-1;
            if (compareAndSetState(c, nextc))
                return nextc == 0;
        }
    }
}
```

## 构造方法
构造方法接收一个int类型的参数作为计数器，并初始化自定义的同步器。
```java
public CountDownLatch(int count) {
    if (count < 0) throw new IllegalArgumentException("count < 0");
    this.sync = new Sync(count);
}
```

## await()
```java
public void await() throws InterruptedException {
    sync.acquireSharedInterruptibly(1);
}
```
`acquireSharedInterruptibly()`的作用是获取共享锁。
如果当前线程是中断状态，则抛出异常`InterruptedException`。否则，调用`tryAcquireShared(arg)`尝试获取共享锁；尝试成功则返回，否则就调用`doAcquireSharedInterruptibly()`。`doAcquireSharedInterruptibly()`会使当前线程一直等待，直到当前线程获取到共享锁(或被中断)才返回。

## countDown()
```java
public void countDown() {
    sync.releaseShared(1);
}
```
`releaseShared()`目的是让当前线程释放锁，首先会通过`tryReleaseShared()`去尝试释放共享锁。尝试成功，则直接返回；尝试失败，则通过`doReleaseShared()`去释放共享锁。

# 实例
老板等几个工人做完事后再来检验工作情况，每个工作进行一次`countDown()`，老板则执行`await()`进行等待。

可以有多个老板同时`await()`一起完成检验工作，这个时候需要用到**共享锁**。