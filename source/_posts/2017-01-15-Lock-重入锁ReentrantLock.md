---
title: 'Lock: 重入锁ReentrantLock'
date: 2017-01-15 10:08:54
categories:
- Java锁
tags:
- 线程
- 锁
- Lock
- ReentrantLock
---

重入锁ReentrantLock，表示该锁能够支持一个线程对资源的重复加锁。此外，还支持获取锁时的公平和非公平性选择，默认为非公平性的。

> synchronized关键字隐式的支持重进入。

# 实现重进入
重进入是指任意线程在获取到锁之后能够再次获取该锁而不会被锁所阻塞。
该特性要解决两个问题：
1. 线程再次获取锁
	锁需要去识别获取锁的线程是否为当前占据锁的线程。如果是，则再次成功获取。
2. 锁的最终释放
	线程重复n次获取了锁，随后在第n次释放该锁后，其他线程能够获取到该锁。锁的最终释放要求，对于获取时进行计数自增，对于释放时进行计数自减。

> ReentrantLock通过组合自定义的同步器来实现锁的获取与释放。

## 获取同步状态
以非公平性实现为例。

该方法增加了**再次获取同步状态**的处理逻辑：通过判断当前线程是否为获取锁的线程来决定获取操作是否成功，如果是获取锁的线程再次请求，则将同步状态值进行增加并返回true，表示获取同步状态成功。

```java
/**
 * Performs non-fair tryLock.  tryAcquire is implemented in
 * subclasses, but both need nonfair try for trylock method.
 */
final boolean nonfairTryAcquire(int acquires) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
}
```

## 释放同步状态
如果该锁被获取了n次，那么前`(n-1)`次`tryRelease(int releases)`方法必须返回false，而只有同步状态完全释放了，才能返回true。
```java
protected final boolean tryRelease(int releases) {
        int c = getState() - releases;
        if (Thread.currentThread() != getExclusiveOwnerThread())
            throw new IllegalMonitorStateException();
        boolean free = false;
        if (c == 0) {
            free = true;
            setExclusiveOwnerThread(null);
        }
        setState(c);
        return free;
    }
```

# 获取锁的公平性问题
锁获取的公平性问题：公平的获取锁，就是在绝对时间上，先对锁进行获取的请求一定先被满足，也可以说**锁获取是顺序的**（FIFO）。反之，是不公平的。

ReentrantLock中分别实现了`FairSync`和`NonFairSync`两种哦同步器实现，对于实现公平性和非公平性的主要区别在于：在获取同步状态的时候公平性同步器实现中判断条件多了`hasQueuedPredecessors()`方法，即加入了同步队列中当前节点是否有前驱节点的判断，如果有前驱节点则表示有线程比当前线程更早地请求获取锁，因此需要等待前驱线程获取并释放锁后才能继续获取锁。

**公平性锁和非公平性锁对比**：
1. 公平性锁保证了锁的获取按照FIFO原则，而代价是进行大量的线程切换。
2. 非公平性锁虽然可能造成线程“饥饿”，但是极少的线程切换，保证了其更大的吞吐量。

> 考虑到刚释放锁的线程再次获取同步状态的几率非常大，非公平性锁会出现同一线程连续获取锁的情况；而公平锁保证每次都是同步队列中的第一个节点获取到锁，因而存在大量的线程切换。

```java
static final class NonfairSync extends Sync {
    /**
     * Performs lock.  Try immediate barge, backing up to normal
     * acquire on failure.
     */
    final void lock() {
        if (compareAndSetState(0, 1))
            setExclusiveOwnerThread(Thread.currentThread());
        else
            acquire(1);
    }
}

static final class FairSync extends Sync {
    final void lock() {
        acquire(1);
    }
}
```
**非公平锁**的lock方法的处理方式是: 在lock的时候先直接cas修改一次state变量（尝试获取锁），成功就返回，不成功再排队，从而达到不排队直接抢占的目的。
而对于**公平锁**：则是老老实实的开始就走AQS的流程排队获取锁。如果前面有人调用过其lock方法，则排在队列中前面，也就更有机会更早的获取锁，从而达到“公平”的目的。