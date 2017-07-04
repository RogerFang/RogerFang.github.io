---
title: 'Java集合框架(12): TreeSet源码分析'
date: 2016-12-30 12:15:37
categories:
- Java集合框架
tags:
- Set
- TreeSet
---

```java
public class TreeSet<E> extends AbstractSet<E>
    implements NavigableSet<E>, Cloneable, java.io.Serializable
```

# 简介
`TreeSet`是基于`TreeMap`实现的，它的作用是提供有序的Set集合，其中的元素支持两种排序方式：**自然排序**和`Comparator`方式。

`TreeSet`为基本操作(add、remove和contains)提供受保证的复杂度$O(\log{N})$。是非线程安全的。

# 源码分析
## 属性
```java
	// The backing map
    private transient NavigableMap<E,Object> m;
    // Dummy value to associate with an Object in the backing Map
    private static final Object PRESENT = new Object();
```

## 构造方法
```java
/**
 * default修饰，不对外公开的
 */
TreeSet(NavigableMap<E,Object> m) {
    this.m = m;
}

/**
 * 默认底层实现使用TreeMap
 */
public TreeSet() {
    this(new TreeMap<E,Object>());
}

public TreeSet(Comparator<? super E> comparator) {
    this(new TreeMap<>(comparator));
}

public TreeSet(Collection<? extends E> c) {
    this();
    addAll(c);
}

public TreeSet(SortedSet<E> s) {
    this(s.comparator());
    addAll(s);
}
```

## 方法
### 迭代器
```java
/**
 * 返回TreeSet的顺序排列的迭代器
 * Returns an iterator over the elements in this set in ascending order.
 */
public Iterator<E> iterator() {
    return m.navigableKeySet().iterator();
}

/**
 * 返回TreeSet的逆序排列的迭代器
 * Returns an iterator over the elements in this set in descending order.
 */
public Iterator<E> descendingIterator() {
    return m.descendingKeySet().iterator();
}
```

### contains
```java
public boolean contains(Object o) {
    return m.containsKey(o);
}
```

### add
```java
public boolean add(E e) {
    return m.put(e, PRESENT)==null;
}
```

### remove
```java
public boolean remove(Object o) {
    return m.remove(o)==PRESENT;
}
```
