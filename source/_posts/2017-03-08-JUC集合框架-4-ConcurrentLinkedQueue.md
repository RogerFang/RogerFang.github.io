---
title: 'JUC集合框架(4): ConcurrentLinkedQueue'
date: 2017-03-08 13:15:37
categories:
- JUC集合框架
tags:
- ConcurrentLinkedQueue
---

`ArrayBlockingQueue`和`LinkedBlockingQueue`是并发队列的阻塞算法实现，而`ConcurrentLinkedQueue`是并发队列的非阻塞算法实现。

```java
public class ConcurrentLinkedQueue<E> extends AbstractQueue<E>
        implements Queue<E>, java.io.Serializable
```

`ConcurrentLinkedQueue`内部使用大量的 **循环CAS** 操作来保证线程安全。

# 属性
一般来说，我们期望 tail 节点总是为链表的尾节点，但实际上，tail 的更新并不是及时的，可能会产生拖延现象。
```java
private transient volatile Node<E> head;
private transient volatile Node<E> tail;
```

# 节点
```java
private static class Node<E> {
        volatile E item;
        volatile Node<E> next;

        ...
}
```

# 入队
入队操作主要完成两件事：第一个是将入队节点设置成当前队列的尾节点；第二个是更新 tail 节点。

对于判断条件`p==q`，这种情况是由于出现了**哨兵节点（sentinel）**导致的。哨兵节点就是 **next** 指向自己的节点，这类节点在队列中存在的价值不大，主要表示删除的节点或空节点。

当遇到**哨兵节点**时，无法通过 next 取得后续的节点，因此很可能直接返回 head，期望从链表头部开始遍历，进一步查找定位链表的尾节点。
但是如果发生在执行过程中，tail 被其他线程修改的情况，则进行一次“打赌”，使用新的 tail 作为链表末尾。

`t != (t = tail)`并不是原子操作，并发环境下右边的t会被更改。

```java
public boolean offer(E e) {
    checkNotNull(e);
    final Node<E> newNode = new Node<E>(e);

    for (Node<E> t = tail, p = t;;) {
        Node<E> q = p.next;
        if (q == null) {
            // p代表链表的最后一个节点
            if (p.casNext(null, newNode)) {
                // 每2次入队操作，更新一下tail节点
                if (p != t) // hop two nodes at a time
                    casTail(t, newNode);  // Failure is OK.
                return true;
            }
            // Lost CAS race to another thread; re-read next
        }
        else if (p == q)
            // We have fallen off list.  If tail is unchanged, it
            // will also be off-list, in which case we need to
            // jump to head, from which all live nodes are always
            // reachable.  Else the new tail is a better bet.
            p = (t != (t = tail)) ? t : head;
        else
            // Check for tail updates after two hops.
            // 取下一个节点或者最后一个节点
            p = (p != t && t != (t = tail)) ? t : q;
    }
}
```

# 出队
出队列就是从队列返回一个元素，并清空该节点对元素的引用。

**并不是每次出队都会更新 head 节点。**

当 head 节点里有元素时，直接弹出 head 节点中的元素，而不会更新 head 节点；只有当 head 节点里没有元素时，出队操作才会更新 head 节点。（这点和 tail 更新类似，**都是通过减少 CAS 更新节点的消耗，从而提高效率。**）

```java
public E poll() {
    restartFromHead:
    for (;;) {
        for (Node<E> h = head, p = h, q;;) {
            E item = p.item;

            if (item != null && p.casItem(item, null)) {
                // Successful CAS is the linearization point
                // for item to be removed from this queue.
                if (p != h) // hop two nodes at a time
                    updateHead(h, ((q = p.next) != null) ? q : p);
                return item;
            }
            else if ((q = p.next) == null) {
                updateHead(h, p);
                return null;
            }
            else if (p == q)
                continue restartFromHead;
            else
                p = q;
        }
    }
}
```