---
title: 'Java集合框架(9): IdentityHashMap源码分析'
date: 2016-12-29 20:55:17
categories:
- Java集合框架
tags:
- Map
- IdentityHashMap
---

```java
public class IdentityHashMap<K,V>
    extends AbstractMap<K,V>
    implements Map<K,V>, java.io.Serializable, Cloneable
```

# 简介
`IdentityHashMap`在平时并不常用，通过例子来简单感受下。
> `IdentityHashMap`和`HashMap`没有任何关系，它俩使用的数据结构完全不同。

```java
public class IdentityHashMapTest {

    public static void main(String[] args) {
        Map<String, String> hashMap = new HashMap<>();
        hashMap.put(new String("a"), "aa");
        hashMap.put(new String("a"), "bb");
        System.out.println(hashMap.size() + ":" + hashMap);
        Map<String, String> identityHashMap = new IdentityHashMap<>();
        identityHashMap.put(new String("a"), "aa");
        identityHashMap.put(new String("a"), "bb");
        System.out.println(identityHashMap.size() + ":" + identityHashMap);
    }
}
```
运行结果如下：
```
1:{a=bb}
2:{a=aa, a=bb}
```
说明：IdentityHashMap只有在key是**同一个引用**时才会覆盖，而HashMap则不会。
> `IdentityHashMap` using **reference-equality** in place of object-equality when comparing keys (and values)

* reference-equality: `k1 == k2`
* object-equality: `k1==null ? k2==null : k1.equals(k2)`

1. `IdentityHashMap`的key和value都可以为null
2. 无序的
3. 非线程安全的

## 数据结构
`IdentityHashMap`的数据结构很简单，**底层就是一个Object数组，在逻辑上可以看成是一个环形的数组**， 没有节点类型，**数组的偶数位存放key，奇数位存放value**。
> 容量为16的哈希表实际申请数组大小为32，当元素的个数(11)大于数组大小(32)的三分之一时扩容。

解决冲突的办法是**开放定址法**：也就是根据计算得到散列的位置索引，如果发现该位置上已有元素，则往后查找，直到找到空位置。当元素个数达到一定阈值时，Object数组会进行扩容。

# 源码分析
## 属性
```java
/**
 * The value 32 corresponds to the (specified) expected
 * maximum size of 21, given a load factor of 2/3.
 */
// 默认容量，大小为2的幂次方
private static final int DEFAULT_CAPACITY = 32;
// 最小容量
private static final int MINIMUM_CAPACITY = 4;
// 最大容量
private static final int MAXIMUM_CAPACITY = 1 << 29;
// 实际存储元素的数组
transient Object[] table; // non-private to simplify nested class access
// 元素的数量
int size;
// 表结构修改的次数
transient int modCount;
// null key所对应的的值(Use NULL_KEY for key if it is null.)
static final Object NULL_KEY = new Object();
```

## 构造函数
```java
/**
 * Constructs a new, empty identity hash map with a default expected
 * maximum size (21).
 */
public IdentityHashMap() {
    init(DEFAULT_CAPACITY);
}

public IdentityHashMap(int expectedMaxSize) {
    if (expectedMaxSize < 0)
        throw new IllegalArgumentException("expectedMaxSize is negative: "
                                           + expectedMaxSize);
    init(capacity(expectedMaxSize));
}

public IdentityHashMap(Map<? extends K, ? extends V> m) {
    // Allow for a bit of growth
    this((int) ((1 + m.size()) * 1.1));
    putAll(m);
}
```

## 方法
### capacity
此函数返回的是最小的且大于expectedMaxSize的2次幂数
```java
private static int capacity(int expectedMaxSize) {
    // assert expectedMaxSize >= 0;
    return
        (expectedMaxSize > MAXIMUM_CAPACITY / 3) ? MAXIMUM_CAPACITY :
        (expectedMaxSize <= 2 * MINIMUM_CAPACITY / 3) ? MINIMUM_CAPACITY :
        Integer.highestOneBit(expectedMaxSize + (expectedMaxSize << 1));
}
```

### hash
由于length总是2的幂次方，所以`& (length - 1)`相当于取模运算。
这个hash方法得到的值始终都是偶数，以保证key始终会存放在偶数位置，而不会插入到value的位置上，value都是放在奇数位置上的。
```java
/**
 * Returns index for Object x.
 */
private static int hash(Object x, int length) {
    int h = System.identityHashCode(x);
    // Multiply by -127, and left-shift to use least bit as part of hash
    return ((h << 1) - (h << 8)) & (length - 1);
}
```

### get
```java
public V get(Object key) {
	// 保证null的key会转化为Object(NULL_KEY)
    Object k = maskNull(key);
    Object[] tab = table;
    int len = tab.length;
    int i = hash(k, len);
    // 遍历table，若冲突则往后寻找空闲区域
    while (true) {
        Object item = tab[i];
        // 判断是否相等
        if (item == k)
        	// 相等，即完全相等的两个对象
            return (V) tab[i + 1];
        if (item == null)
            return null;
        // 取下一个Key索引
        i = nextKeyIndex(i, len);
    }
}
```

### nextKeyIndex
下一个Key的索引，用于发生冲突时，取下一个位置进行判断。
nextKeyIndex始终加2，如果key在当前位置已经存在值，那么继续寻找下一个位置，这个时候就会在此位置的基础上加2，如果加1就会和value冲突了。
```java
private static int nextKeyIndex(int i, int len) {
    return (i + 2 < len ? i + 2 : 0);
}
```

### put
```java
public V put(K key, V value) {
	// 保证null的key会转化为Object(NULL_KEY)
    final Object k = maskNull(key);

    retryAfterResize: for (;;) {
        final Object[] tab = table;
        final int len = tab.length;
        int i = hash(k, len);

        for (Object item; (item = tab[i]) != null;
             i = nextKeyIndex(i, len)) {
            if (item == k) {
                @SuppressWarnings("unchecked")
                    V oldValue = (V) tab[i + 1];
                tab[i + 1] = value;
                return oldValue;
            }
        }

        final int s = size + 1;
        // Use optimized form of 3 * s.
        // Next capacity is len, 2 * current capacity.
        // 如果3*size大于length，则会进行扩容
        if (s + (s << 1) > len && resize(len))
        	// 扩容后重新计算元素的位置索引，寻找合适的位置进行存放
            continue retryAfterResize;

        modCount++;
        tab[i] = k;
        tab[i + 1] = value;
        size = s;
        return null;
    }
}
```

### resize
```java
private boolean resize(int newCapacity) {
    // assert (newCapacity & -newCapacity) == newCapacity; // power of 2
    int newLength = newCapacity * 2;

    Object[] oldTable = table;
    int oldLength = oldTable.length;
    // 无法继续扩容
    if (oldLength == 2 * MAXIMUM_CAPACITY) { // can't expand any further
        if (size == MAXIMUM_CAPACITY - 1)
            throw new IllegalStateException("Capacity exhausted.");
        return false;
    }
    // 旧表长度大于新表长度，不进行扩容
    if (oldLength >= newLength)
        return false;

    Object[] newTable = new Object[newLength];

	// 生产新表，将旧表元素重新hash到新表中
    for (int j = 0; j < oldLength; j += 2) {
        Object key = oldTable[j];
        if (key != null) {
            Object value = oldTable[j+1];
            oldTable[j] = null;
            oldTable[j+1] = null;
            int i = hash(key, newLength);
            while (newTable[i] != null)
                i = nextKeyIndex(i, newLength);
            newTable[i] = key;
            newTable[i + 1] = value;
        }
    }
    table = newTable;
    return true;
}
```

### remove
```java
public V remove(Object key) {
    Object k = maskNull(key);
    Object[] tab = table;
    int len = tab.length;
    int i = hash(k, len);

    while (true) {
        Object item = tab[i];
        // 找到key相等的项
        if (item == k) {
            modCount++;
            size--;
            @SuppressWarnings("unchecked")
                V oldValue = (V) tab[i + 1];
            tab[i + 1] = null;
            tab[i] = null;
            // 删除后需要进行后续处理，把之前由于冲突往后挪的元素移到前面来
            closeDeletion(i);
            return oldValue;
        }
        
        // 遍历到项为null时，结束遍历
        if (item == null)
            return null;
        // 下一项
        i = nextKeyIndex(i, len);
    }
}

private void closeDeletion(int d) {
    // Adapted from Knuth Section 6.4 Algorithm R
    Object[] tab = table;
    int len = tab.length;

    // Look for items to swap into newly vacated slot
    // starting at index immediately following deletion,
    // and continuing until a null slot is seen, indicating
    // the end of a run of possibly-colliding keys.
    // 把该元素后面符合移动规定的元素往前面移动
    Object item;
    for (int i = nextKeyIndex(d, len); (item = tab[i]) != null;
         i = nextKeyIndex(i, len) ) {
        // The following test triggers if the item at slot i (which
        // hashes to be at slot r) should take the spot vacated by d.
        // If so, we swap it in, and then continue with d now at the
        // newly vacated i.  This process will terminate when we hit
        // the null slot at the end of this run.
        // The test is messy because we are using a circular table.
        int r = hash(item, len);
        if ((i < r && (r <= d || d <= i)) || (r <= d && d <= i)) {
            tab[d] = item;
            tab[d + 1] = tab[i + 1];
            tab[i] = null;
            tab[i + 1] = null;
            d = i;
        }
    }
}
```

* * *
感谢：
http://www.cnblogs.com/leesf456/p/5253094.html
http://www.itdadao.com/articles/c15a645336p0.html