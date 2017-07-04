---
title: 'Java集合框架(10): HashSet源码分析'
date: 2016-12-30 12:15:01
categories:
- Java集合框架
tags:
- Set
- HashSet
---

```java
public class HashSet<E>
    extends AbstractSet<E>
    implements Set<E>, Cloneable, java.io.Serializable
```

# 简介
`Set`接口扩展自`Collection`接口，规定`Set`的实例不包含重复的元素。

1. `HashSet`是一个没有重复元素的集合。
2. 不保证元素的顺序，允许使用null元素。
3. 非线程安全的。

`HashSet`是基于`HashMap`来实现的，底层采用`HashMap`来保存元素。

# 源码分析
## 属性
```java
	// 底层使用HashMap来保存HashSet的元素
    private transient HashMap<E,Object> map;
    // 由于Set只使用了HashMap的key，所以此处定义一个静态的常量Object类，来充当HashMap的vlaue
    private static final Object PRESENT = new Object();
```
这里的`PRESENT`使用`new Object()`来充当value，而不直接使用`null`，目的就是从根源上避免`NullPointerException`，代码中也不需要取校验value是否为空。

## 构造方法
```java
public HashSet() {
    map = new HashMap<>();
}

public HashSet(Collection<? extends E> c) {
    map = new HashMap<>(Math.max((int) (c.size()/.75f) + 1, 16));
    addAll(c);
}

public HashSet(int initialCapacity, float loadFactor) {
    map = new HashMap<>(initialCapacity, loadFactor);
}

public HashSet(int initialCapacity) {
    map = new HashMap<>(initialCapacity);
}

/**
 * 这个构造方法比较特殊，默认default修饰，不对外公开
 * only used by LinkedHashSet
 * 底层构造的是LinkedHashMap，dummy只是一个标示参数，无具体意义
 */
HashSet(int initialCapacity, float loadFactor, boolean dummy) {
    map = new LinkedHashMap<>(initialCapacity, loadFactor);
}
```

## 方法
### 迭代
返回`HashMap`的键迭代器
```java
public Iterator<E> iterator() {
    return map.keySet().iterator();
}
```

### contains
```java
public boolean contains(Object o) {
    return map.containsKey(o);
}
```

### add
当set不存在这个元素时添加成功返回true，否则返回false
```java
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}
```

### remove
```java
public boolean remove(Object o) {
    return map.remove(o)==PRESENT;
}
```

### clone
```java
/**
 * 浅拷贝
 */
public Object clone() {
    try {
        HashSet<E> newSet = (HashSet<E>) super.clone();
        newSet.map = (HashMap<E, Object>) map.clone();
        return newSet;
    } catch (CloneNotSupportedException e) {
        throw new InternalError(e);
    }
}
```

* * *
感谢：
http://www.cnblogs.com/leesf456/p/5309809.html
http://www.jianshu.com/p/c5f85e9c0098