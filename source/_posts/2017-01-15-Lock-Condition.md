---
title: 'Lock: Condition'
date: 2017-01-15 13:22:49
categories:
- Java锁
tags:
- 线程
- 锁
- Lock
- Condition
---

任意一个Java对象，都拥有一组监视器方法（定义在java.lang.Object上），主要包括`wait()`、`wait(long timeout)`、`notify()`、`notifyAll()`，这些方法与`synchronized`关键字配合，可以实现**等待/通知机制**。

Condition的作用是对锁进行更精确的控制，也提供了类似Object的监视器方法。
Condition中的`await()`方法相当于Object的`wait()`方法，Condition中的`signal()`方法相当于Object的`notify()`方法，Condition中的`signalAll()`相当于Object的`notifyAll()`方法。

> 不同的是，**Object中的监视器方法是和"同步锁"(synchronized关键字)配合使用的**；而`Condition`**是需要与"排他锁"/"共享锁" Lock 配合使用的**。

# Condition实现分析
ConditionObject是队列同步器AbstractQueuedSynchronizer的内部类，实现了Condition接口。因为Condition的操作需要获取相关联的锁，所以作为同步器的内部类也较为合理。
```java
public class ConditionObject implements Condition, java.io.Serializable
```
Condition的实现主要包括：等待队列、等待、通知。
## 等待队列
等待队列是一个FIFO的队列，在队列中的每个节点都包含了一个线程引用，
> 等待队列的节点复用了同步器中同步队列的节点定义`AbstractQueuedSynchronizer.Node`。

一个Condition包含一个等待队列，Condition拥有首节点（firstWaiter）和尾节点（lastWaiter）。
尾节点的更新并没有像同步队列中使用CAS保证，原因在于调用`await()`方法的线程必定是获取了锁的线程，也就是说该过程是由锁来保证线程安全的。

> 在Object监视器模型上，一个对象拥有一个同步队列和一个等待队列；而并发包中的Lock拥有一个同步队列和多个等待队列。

## 等待
调用Condition的`await()`方法会使当前线程进入等待队列并释放锁，同时线程状态会变为等待状态。从队列的角度看，相当于同步队列的首节点（获取了锁的节点）移动到Condition的等待队列中（通过`addConditionWaiter()`方法把当前线程构造成一个新的节点）。
```java
public final void await() throws InterruptedException {
        if (Thread.interrupted())
            throw new InterruptedException();
        // 当前线程加入到等待队列
        Node node = addConditionWaiter();
        // 释放同步状态(锁)
        int savedState = fullyRelease(node);
        int interruptMode = 0;
        // 循环处理被唤醒的过程
        while (!isOnSyncQueue(node)) {
            LockSupport.park(this);
            if ((interruptMode = checkInterruptWhileWaiting(node)) != 0)
                break;
        }
        if (acquireQueued(node, savedState) && interruptMode != THROW_IE)
            interruptMode = REINTERRUPT;
        if (node.nextWaiter != null) // clean up if cancelled
            unlinkCancelledWaiters();
        if (interruptMode != 0)
            reportInterruptAfterWait(interruptMode);
    }
```
当等待队列中的节点被**唤醒**，则唤醒的节点尝试获取同步状态。如果不是通过其他线程调用`signal()`方法唤醒，而是对等待线程进行中断，则会抛出异常`InterruptedException`。

## 通知
调用Condition的`signal()`方法会唤醒等待队列中等待时间最长的节点（首节点），在唤醒节点之前，会将节点移到同步队列中。
```java
public final void signal() {
	// 前置条件必须是获取了锁，在此进行检查
    if (!isHeldExclusively())
        throw new IllegalMonitorStateException();
    Node first = firstWaiter;
    if (first != null)
        doSignal(first);
}
```
**唤醒的过程**：
1. 调用该方法的前置条件是当前线程必须获取了锁
2. 接着获取等待队列的首节点，调用同步器的`enq(Node node)`方法将其移动到同步队列中
3. 最后使用 LockSupport 唤醒节点中的线程。
4. 被唤醒的额线程将从`await()`方法中的while循环中退出，进而调用同步器的`acquireQueued()`方法加入到获取同步状态的竞争中。
5. 成功获取同步状态之后，被唤醒的线程将从先前调用的`await()`方法返回，此时已经成功获取锁。

> Condition的`signalAll()`方法，相当于对等待队列中的每个节点均执行一次`signal()`方法，效果就是将等待队列中的所有节点全部移动到同步队列中，并唤醒每个节点的线程。