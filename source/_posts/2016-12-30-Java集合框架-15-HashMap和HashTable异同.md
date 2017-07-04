---
title: 'Java集合框架(15): HashMap和HashTable异同'
date: 2016-12-30 17:08:25
categories:
- Java集合框架
tags:
- Map
- HashMap
- HashTable
---

# 相同点
> `HashMap`和`HashTable`都是存储“键值对”的散列表，都采用拉链法解决Hash冲突。

**添加**key-value键值对：首先，根据key值计算出哈希值，再计算出数组索引(即，该key-value在table中的索引)。然后，根据数组索引找到Entry(即，单向链表)，再遍历单向链表，将key和链表中的每一个节点的key进行对比。若key已经存在Entry链表中，则用该value值取代旧的value值；若key不存在Entry链表中，则新建一个key-value节点，并将该节点插入Entry链表的表头位置。

**删除**key-value键值对：删除键值对，相比于“添加键值对”来说，简单很多。首先，还是根据key计算出哈希值，再计算出数组索引(即，该key-value在table中的索引)。然后，根据索引找出Entry(即，单向链表)。若节点key-value存在与链表Entry中，则删除链表中的节点即可。

# 不同点
## 继承和实现方式不同
`HashMap` 继承于AbstractMap，实现了Map、Cloneable、java.io.Serializable接口。
`Hashtable` 继承于Dictionary，实现了Map、Cloneable、java.io.Serializable接口。

## 线程安全不同
`HashTable`是线程安全的，几乎所有函数都是同步的。
`HashMap`不是线程安全的。可以使用`Collections.synchronizedMap`或者java.util.concurrent包里的`ConcurrentHashMap`类。

## null处理
`HashMap`的key、value都可以为null。
`HashTable`的key、value都不可以为null。当HashMap的key为null时，HashMap会将其固定的插入table[0]位置(即HashMap散列表的第一个位置)；而且table[0]处只会容纳一个key为null的值。

## 遍历支持不同
`HashMap`只支持Iterator(迭代器)遍历。
`Hashtable`支持Iterator(迭代器)和Enumeration(枚举器)两种方式遍历。

`Enumeration` 是JDK 1.0添加的接口，只有hasMoreElements(), nextElement() 两个API接口，不能通过Enumeration()对元素进行修改 。
`Iterator` 是JDK 1.2才添加的接口，支持hasNext(), next(), remove() 三个API接口。HashMap也是JDK 1.2版本才添加的，所以用Iterator取代Enumeration，HashMap只支持Iterator遍历。

## Iterator迭代器遍历顺序不同
`HashMap`是“从前向后”的遍历数组；再对数组具体某一项对应的链表，从表头开始进行遍历。
`Hashtable`是“从后往前”的遍历数组；再对数组具体某一项对应的链表，从表头开始进行遍历。

## 初始容量不同
`HashMap`默认的容量大小是16
`Hashtable`默认的容量大小是11

## 扩容方式不同
`HashMap`增加容量时，每次将容量变为“原始容量x2”。
扩容时，会将单链表拆分为高低两个链表（低链表还是保存在原索引，高链表保存在原索引+oldCap的位置）。

`HashTable`增加容量时，每次将容量变为“原始容量x2 + 1”。
扩容时，会将每个元素遍历一遍重新计算hash值得到索引位置。

## hash算法不同
`HashMap` 使用自定义的哈希算法，`(key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16)`。

`HashTable` 则是`(key.hashCode() & 0x7FFFFFFF)`。

## 部分API不同
`HashMap`没有重写`toString()`方法。
`HashTable`重写了`toString()`方法。

```java HashTable
public synchronized String toString() {
    int max = size() - 1;
    if (max == -1)
        return "{}";

    StringBuilder sb = new StringBuilder();
    Iterator<Map.Entry<K,V>> it = entrySet().iterator();

    sb.append('{');
    for (int i = 0; ; i++) {
        Map.Entry<K,V> e = it.next();
        K key = e.getKey();
        V value = e.getValue();
        sb.append(key   == this ? "(this Map)" : key.toString());
        sb.append('=');
        sb.append(value == this ? "(this Map)" : value.toString());

        if (i == max)
            return sb.append('}').toString();
        sb.append(", ");
    }
}
```

* * *
感谢：
http://wangkuiwu.github.io/2012/02/14/collection-14-mapsummary/
