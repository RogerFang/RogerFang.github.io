---
title: 'JUC集合框架(5): CopyOnWriteArrayList'
date: 2017-03-08 13:52:17
categories:
- JUC集合框架
tags:
- CopyOnWriteArrayList
---

> 适用于读操作远大于写操作的场景。

`CopyOnWriteArrayList`的读操作不会加锁，写操作也不会阻塞读操作，只有写和写操作之间需要同步等待。

```java
public class CopyOnWriteArrayList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

# 属性
```java
/** The lock protecting all mutators */
final transient ReentrantLock lock = new ReentrantLock();

/** The array, accessed only via getArray/setArray. */
private transient volatile Object[] array;
```

# 读
`CopyOnWriteArrayList`的读操作代码完全没有任何同步控制和锁操作，因为内部数组 array 不会发生修改，只会被另外一个 array 替换掉，因此可以保证数据安全。

```java
public E get(int index) {
    return get(getArray(), index);
}

final Object[] getArray() {
    return array;
}

private E get(Object[] a, int index) {
    return (E) a[index];
}
```

# 写
`CopyOnWriteArrayList`的写操作需要使用锁，这个锁仅用于控制**写-写**的情况。在进行写操作时，会进行一次自我复制，将修改的内容写入 array 的副本，写完后再替换原来的 array（array 是volatile关键字修饰的）。

```java
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();
    try {
        Object[] elements = getArray();
        int len = elements.length;
        Object[] newElements = Arrays.copyOf(elements, len + 1);
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();
    }
}
```