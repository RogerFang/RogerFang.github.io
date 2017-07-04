---
title: 'Java集合框架(11): LinkedHashSet源码分析'
date: 2016-12-30 12:15:20
categories:
- Java集合框架
tags:
- Set
- LinkedHashSet
---

```java
public class LinkedHashSet<E>
    extends HashSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
```

# 简介
`LinkedHashSet`继承自`HashSet`，`LinkedHashSet`调用父类`HashSet`的构造函数(default修饰)，其底层实现使用的map是`LinkedHashMap`。

`LinkedHashSet`的元素可以按照**插入顺序**来访问。

```java
/**
 * Constructs a new, empty linked hash set.  (This package private
 * constructor is only used by LinkedHashSet.) The backing
 * HashMap instance is a LinkedHashMap with the specified initial
 * capacity and the specified load factor.
 *
 * @param      initialCapacity   the initial capacity of the hash map
 * @param      loadFactor        the load factor of the hash map
 * @param      dummy             ignored (distinguishes this
 *             constructor from other int, float constructor.)
 * @throws     IllegalArgumentException if the initial capacity is less
 *             than zero, or if the load factor is nonpositive
 */
HashSet(int initialCapacity, float loadFactor, boolean dummy) {
    map = new LinkedHashMap<>(initialCapacity, loadFactor);
}
```