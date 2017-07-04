---
title: 'Java集合框架(4): Vector源码分析'
date: 2016-12-28 09:53:32
categories:
- Java集合框架
tags:
- Vector
---

```java
public class Vector<E>
    extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

# 简介
1. Vector和ArrayList一样是基于**数组**实现的，是一个**动态**数组，其容量能自动增长。
2. Vector和ArrayList的区别在于，Vector是**线程安全**的。
3. 和ArrayList实现了一样的接口

# 源码分析
## 属性
```java
	// 用该数组保存所有元素
    protected Object[] elementData;
    // 元素的数量
    protected int elementCount;
    // 每次扩容时的增量
    protected int capacityIncrement;
```

## 构造方法
```java
public Vector(int initialCapacity, int capacityIncrement) {
    super();
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    this.elementData = new Object[initialCapacity];
    this.capacityIncrement = capacityIncrement;
}

public Vector(int initialCapacity) {
    this(initialCapacity, 0);
}

public Vector() {
    this(10);
}

public Vector(Collection<? extends E> c) {
    elementData = c.toArray();
    elementCount = elementData.length;
    // c.toArray might (incorrectly) not return Object[] (see 6260652)
    if (elementData.getClass() != Object[].class)
        elementData = Arrays.copyOf(elementData, elementCount, Object[].class);
}
```

## 方法
### 扩容
```java
public synchronized void ensureCapacity(int minCapacity) {
    if (minCapacity > 0) {
        modCount++;
        ensureCapacityHelper(minCapacity);
    }
}

private void ensureCapacityHelper(int minCapacity) {
    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        grow(minCapacity);
}

private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                     capacityIncrement : oldCapacity);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    elementData = Arrays.copyOf(elementData, newCapacity);
}

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
        MAX_ARRAY_SIZE;
}
```
> Vector：如果在构造函数中没有指定每次扩容时应该增加的容量大小，则每次扩容默认为容量加倍。
> ArrayList对比：ArrayList每次扩容则是默认增加0.5倍左右。

### 基本方法
```java
public synchronized void setSize(int newSize) {
    modCount++;
    if (newSize > elementCount) {
        ensureCapacityHelper(newSize);
    } else {
        for (int i = newSize ; i < elementCount ; i++) {
            elementData[i] = null;
        }
    }
    elementCount = newSize;
}

public synchronized int size() {
    return elementCount;
}

public synchronized boolean add(E e) {
    modCount++;
    ensureCapacityHelper(elementCount + 1);
    elementData[elementCount++] = e;
    return true;
}

public synchronized boolean removeElement(Object obj) {
    modCount++;
    int i = indexOf(obj);
    if (i >= 0) {
        removeElementAt(i);
        return true;
    }
    return false;
}

public synchronized int indexOf(Object o, int index) {
    if (o == null) {
        for (int i = index ; i < elementCount ; i++)
            if (elementData[i]==null)
                return i;
    } else {
        for (int i = index ; i < elementCount ; i++)
            if (o.equals(elementData[i]))
                return i;
    }
    return -1;
}

public synchronized List<E> subList(int fromIndex, int toIndex) {
    return Collections.synchronizedList(super.subList(fromIndex, toIndex),
                                        this);
}
```
> Vector的线程安全是通过synchronized关键字来实现的，要么是方法上直接添加synchronized关键字，要么是嵌套调用同步的方法。
> Vector也可以包含null元素。

### 迭代器
```java
public synchronized ListIterator<E> listIterator() {
    return new ListItr(0);
}

public synchronized Iterator<E> iterator() {
    return new Itr();
}
```
> Vector包括和ArrayList一样的Iterator和ListIterator迭代器，迭代器内部的方法也是同步的。同时还有独特的迭代方式Enumeration

```java
public Enumeration<E> elements() {
    return new Enumeration<E>() {
        int count = 0;

        public boolean hasMoreElements() {
            return count < elementCount;
        }

        public E nextElement() {
            synchronized (Vector.this) {
                if (count < elementCount) {
                    return elementData(count++);
                }
            }
            throw new NoSuchElementException("Vector Enumeration");
        }
    };
}
```

* * *

感谢：
http://blog.jrwang.me/2016/java-collections-vector-stack/
http://blog.csdn.net/chdjj/article/details/38494183