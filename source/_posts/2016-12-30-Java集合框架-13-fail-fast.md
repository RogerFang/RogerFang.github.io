---
title: 'Java集合框架(13): fail-fast'
date: 2016-12-30 13:23:55
categories:
- Java集合框架
tags:
- fail-fast
---

# 简介
fail-fast机制是java集合框架中的一种错误机制。当多个线程对同一个**集合的结构**进行操作时，可能会产生fail-fast，出现`java.util.ConcurrentModificationException`。

# 解决办法
fail-fast机制，是一种**错误检测机制**，只能被用来检测错误，因为**JDK并不保证fail-fast机制一定发生**。
1. 方法一：在多线程环境下，应该使用“java.util.concurrent”包下的类去取代“java.util”包下的类。
2. 方法二：在遍历过程中，所有涉及改变modCount值得地方全部加上synchronized或直接使用Collections.synchronizedList（但是容器结构的更改操作造成的同步锁，可能会阻塞遍历操作，不推荐）。

# ArrayList和CopyOnWriteArrayList
这里采用`ArrayList`的fail-fast机制和`CopyOnWriteArrayList`来解决这个问题。
## ArrayList
ArrayList的Iterator会采用fail-fast机制抛出异常`ConcurrentModificationException`。
```java
private class Itr implements Iterator<E> {
    ...

    // 容器的结构修改次数
    // 如果遍历时不相等，则抛出ConcurrentModificationException异常，产生fail-fast事件
    int expectedModCount = modCount;

    ...

    final void checkForComodification() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
}
```

## CopyOnWriteArrayList
`CopyOnWriteArrayList`的对容器结构的更改方法（add、remove、clear等，以add方法为例），都是对原底层结构array进行复制后，再在复制的数组上进行更改操作，这样就不会影响它的迭代器`COWIterator`中的array了。
### 结构更改操作
```java
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        // 看这里
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}
```

### 迭代器
`CopyOnWriteArrayList`并不会进行fail-fast机制检测。
```java
static final class COWIterator<E> implements ListIterator<E> {
    /** Snapshot of the array */
    private final Object[] snapshot;
    /** Index of element to be returned by subsequent call to next.  */
    private int cursor;

    private COWIterator(Object[] elements, int initialCursor) {
        cursor = initialCursor;
        snapshot = elements;
    }

    /**
     * Not supported. Always throws UnsupportedOperationException.
     * @throws UnsupportedOperationException always; {@code remove}
     *         is not supported by this iterator.
     */
    public void remove() {
        throw new UnsupportedOperationException();
    }

    /**
     * Not supported. Always throws UnsupportedOperationException.
     * @throws UnsupportedOperationException always; {@code set}
     *         is not supported by this iterator.
     */
    public void set(E e) {
        throw new UnsupportedOperationException();
    }

    /**
     * Not supported. Always throws UnsupportedOperationException.
     * @throws UnsupportedOperationException always; {@code add}
     *         is not supported by this iterator.
     */
    public void add(E e) {
        throw new UnsupportedOperationException();
    }
}
```