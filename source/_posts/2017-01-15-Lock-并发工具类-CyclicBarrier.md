---
title: 'Lock: 并发工具类-CyclicBarrier'
date: 2017-01-15 14:21:14
categories:
- Java锁
tags:
- 线程
- 锁
- Lock
- CyclicBarrier
---

CyclicBarrier可以让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，所有被屏障拦截的线程才会继续运行。

# CountDownLatch和CyclicBarrier的区别
1. CountDownLatch的作用是允许1或N个线程等待其他线程完成执行；而CyclicBarrier则是允许一组线程相互等待。
2. CountDownLatch的计数器无法被重置，只能使用一次；CyclicBarrier的计数器可以被重置后使用，因此它被称为是循环的barrier，能处理更为复杂的业务场景。
3. CountDownLatch通过自定义同步器（共享锁）实现的；而CyclicBarrier是通过重入锁ReentrantLock（排他锁）和Condition来实现的。


# 源码分析
## 依赖
CyclicBarrier是通过重入锁ReentrantLock和Condition实现的。
```java
/** The lock for guarding barrier entry */
private final ReentrantLock lock = new ReentrantLock();
/** Condition to wait on until tripped */
private final Condition trip = lock.newCondition();
```

## 构造方法
```java
public CyclicBarrier(int parties) {
    this(parties, null);
}

public CyclicBarrier(int parties, Runnable barrierAction) {
    if (parties <= 0) throw new IllegalArgumentException();
    // parties表示“必须同时到达barrier的线程个数”
    this.parties = parties;
    // count表示“处在等待状态的线程个数”
    this.count = parties;
    // barrierCommand表示“parties个线程到达barrier时，会优先执行的动作”
    this.barrierCommand = barrierAction;
}
```

## await()
```java
public int await() throws InterruptedException, BrokenBarrierException {
    try {
        return dowait(false, 0L);
    } catch (TimeoutException toe) {
        throw new Error(toe); // cannot happen
    }
}

/**
 * Main barrier code, covering the various policies.
 */
private int dowait(boolean timed, long nanos)
    throws InterruptedException, BrokenBarrierException,
           TimeoutException {
    final ReentrantLock lock = this.lock;
    // 获取“排他锁(lock)”
    lock.lock();
    try {
    	// 保存当前的generation
        final Generation g = generation;

		// 如果当前generation损坏，则抛出异常
        if (g.broken)
            throw new BrokenBarrierException();

		// 如果当前线程被中断，则通过breakBarrier()终止CyclicBarrier，唤醒CyclicBarrier中所有等待线程。
        if (Thread.interrupted()) {
            breakBarrier();
            throw new InterruptedException();
        }

        int index = --count;
        // // 如果index=0，则意味着“有parties个线程到达barrier”。
        if (index == 0) {  // tripped
            boolean ranAction = false;
            try {
                final Runnable command = barrierCommand;
                if (command != null)
                    command.run();
                ranAction = true;
                // 唤醒所有等待线程，并更新generation。
                nextGeneration();
                return 0;
            } finally {
                if (!ranAction)
                    breakBarrier();
            }
        }

        // loop until tripped, broken, interrupted, or timed out
        // 当前线程一直阻塞，直到“有parties个线程到达barrier” 或 “当前线程被中断” 或 “超时”这3者之一发生，
        // 当前线程才继续执行。
        for (;;) {
            try {
                if (!timed)
                    trip.await();
                else if (nanos > 0L)
                    nanos = trip.awaitNanos(nanos);
            } catch (InterruptedException ie) {
                if (g == generation && ! g.broken) {
                    breakBarrier();
                    throw ie;
                } else {
                    // We're about to finish waiting even if we had not
                    // been interrupted, so this interrupt is deemed to
                    // "belong" to subsequent execution.
                    Thread.currentThread().interrupt();
                }
            }

            // 如果“当前generation已经损坏”，则抛出异常。
            if (g.broken)
                throw new BrokenBarrierException();

			// 如果“generation已经换代”，则返回index。
            if (g != generation)
                return index;

			// 如果是“超时等待”，并且时间已到，则通过breakBarrier()终止CyclicBarrier，唤醒CyclicBarrier中所有等待线程。
            if (timed && nanos <= 0L) {
                breakBarrier();
                throw new TimeoutException();
            }
        }
    } finally {
        lock.unlock();
    }
}
```
在CyclicBarrier中，同一批的线程属于同一代，即同一个Generation；CyclicBarrier中通过generation对象，记录属于哪一代。
当有parties个线程到达barrier，generation就会被更新换代。


# 实例
没有老板，几个工人各做各的事，但是每个工人必须在达到 barrier（比如工作时间满了1小时） 时等待其他工人也达到这个 barrier（调用`await()`方法），每个工人再重新开始继续下面的工作。


* * *
感谢：
http://www.cnblogs.com/skywang12345/p/3533995.html
