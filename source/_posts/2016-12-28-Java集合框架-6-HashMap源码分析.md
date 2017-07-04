---
title: 'Java集合框架(6): HashMap源码分析'
date: 2016-12-28 12:28:00
categories:
- Java集合框架
tags:
- Map
- HashMap
---

```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable
```

# 简介
`HashMap`是一种基于哈希表(hash table)实现的map，数组中每一个元素都是一个链表，把数组中的每一格称为一个 bin 或 bucket。

`HashTable` 的实现和 `HashMap` 非常相似，不过 HashTable 对每个方法进行了 Synchronize 同步以保证在多线程场景下的数据一致性，这种实现比较低效。在多线程场景下可以使用 `Map m = Collections.synchronizedMap(new HashMap(...));` 或并发包中的 `ConcurrentHashMap`。

## 设计原理
> 桶 + 链表 + 红黑树

由于每一个桶中都是一个单向链表，hash相同的键值对都会作为一个节点被加入这个链表。**当桶中的键值对数量过多时，会将桶中的单向链表转化为一个树**。通过TREEIFY_THRESHOLD、UNTREEIFY_THRESHOLD和MIN_TREEIFY_CAPACITY来控制转换需要的阈值。

> 在**JDK 8之前**的 HashMap 中都只是采取了单向链表的方式，哈希碰撞会给查找带来灾难性的影响。**在最差的情况下，HashMap 会退化为一个单链表**，查找时间由 O(1) 退化为 O(n) 。而在JDK 8中，如果单链表过长则会转换为一颗红黑树，使得最坏情况下查找的时间复杂度为 O(log n) 。红黑树节点的空间占用相较于普通节点要高出许多，通常只有在比较极端的情况下才会由单链表转化为红黑树。

哈希表（也叫关联数组）一种通用的数据结构。
**哈希表**的概念：key经过hash函数后得到一个槽（buckets或slots）的索引，槽中保存着想要获取的值。伴随哈希表的还有哈希冲突，也就是一些key经过同一hash函数后可能产生相同的索引。在使用哈希表这种数据结构实现具体类时，需要注意：**（1）设计好的hash函数，尽量减少冲突；（2）解决冲突发生后如何处理。**

> **装填因子**（load factor）为散列表中的元素个数对该表大小的比。

## 特点
1. 线程非安全，并且允许key和value为null，HashTable的键值都不能为null。
2. 不保证内部元素的顺序，随着元素增加，同一元素的位置也可能改变（resize的情况）。
3. 遍历集合的时间复杂度与其容量（capacity， 槽的个数）和现有元素的大小（entry的个数）成正比

# 源码分析
## 属性
**默认初始容量（槽的数量）：16，容量必须为2的整数次幂**

```java
	// 默认初始容量（槽的数量）：16，容量必须为2的整数次幂
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
    // 最大容量
    static final int MAXIMUM_CAPACITY = 1 << 30;
    // 默认装填因子
    static final float DEFAULT_LOAD_FACTOR = 0.75f;

	// 树的阈值，当链表长度超过这个值的时候，进行链表到树结构的转变
    static final int TREEIFY_THRESHOLD = 8;
    // 当低于这个值得时候，树转变成链表
    static final int UNTREEIFY_THRESHOLD = 6;
    // 位桶（bin）处的数据要采用红黑树结构进行存储时，整个table的最小容量
    static final int MIN_TREEIFY_CAPACITY = 64;
	// 存储桶的数组，table的长度总是2的幂次方
    transient Node<K,V>[] table;

    transient Set<Map.Entry<K,V>> entrySet;
    // 元素的个数，实际存在的键值对个数
    transient int size;
    // 被修改的次数
    transient int modCount;
    // 临界值（所能容纳的key-value键值对极限），当实际大小（容量*装填因子）超过临界值时，会进行扩容
    int threshold;
    // 装填因子
    final float loadFactor;
```

## 构造方法
HashMap的构造方法默认容量为16，table数组初始化采用了延迟加载的方式，直到第一次调用put方法时才会真正地分配数组空间。

```java
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}

public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
}

public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);
}
```

## 节点
### 链表节点Node
HashMap的Entry定义，其实就是单向链表的一个节点Node，实现了Map.Entry接口，包括了键、值以及下一个节点的引用。这个Node节点是在普通的桶中使用的，在树形桶中使用TreeNode。

```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    V value;
    Node<K,V> next;

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }

    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }

    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
```

### 红黑树节点TreeNode
树形桶中使用的节点`TreeNode`是`LinkedHashMap.Entry`的子类，而`LinkedHashMap.Entry`又是`HashMap.Node`的子类。

使用TreeNode构造一颗红黑树，在红黑树中查找时可以发挥二叉查找的优势。当然，由于TreeNode也可以被当做普通节点Node的扩展，红黑树也可以按照单链表的方式进行遍历。

```java
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    TreeNode<K,V> parent;  // red-black tree links
    TreeNode<K,V> left;
    TreeNode<K,V> right;
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;
    TreeNode(int hash, K key, V val, Node<K,V> next) {
        super(hash, key, val, next);
    }

    /**
     * Returns root of tree containing this node.
     */
    final TreeNode<K,V> root() {
        for (TreeNode<K,V> r = this, p;;) {
            if ((p = r.parent) == null)
                return r;
            r = p;
        }
    }
    ...
}
```

## 方法
### hash算法
确定哈希桶数组的索引位置是关键的一步，通过hash算法求得这个位置。
在计算哈希值时，使用key对象自身的哈希值，并让高位和低位进行异或操作。
jdk1.7，使用indexFor来定位哈希桶数组的索引位置。

```java
/**
 * 方法一 哈希计算
 * jdk1.8&jdk1.7
 */
static final int hash(Object key) {
    int h;
    // 第一步：取key的hashCode
    // 第二步：高位参与运算
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}

/**
 * 方法二 索引映射
 * jdk1.7
 */
static int indexFor(int h, int length) {  //jdk1.7的源码，jdk1.8没有这个方法，但实现原理一样
 return h & (length-1);  //第三步：取模运算
}
```
> HashMap的Hash算法本质上就是三步：取key的hashCode值、高位参与运算、取模运算。

方法一其实是优化单纯使用key的hashCode值实现的Hash算法，其中第二步高位参与运算则是优化的核心。
> 记得在数据结构的《算法》散列表那部分看到过这么一个说法：保证散列均匀性的最好办法也许就是保证键的每一位都在散列值得计算中起到了作用，实现散列函数最常见的错误也许就是忽略了键的高位。

HashMap的取模运算并不是利用`%`来实现的（取模运算消耗比较大），HashMap实现的方法`h& (table.length-1)`来得到相对应得table索引值。HashMap底层数组的长度总是2的n次方，这是HashMap在速度上的优化。当length总是2的n次方时，`h& (table.length-1)`运算等价于对length取模，也就是`h%length`，但是`&`比`%`具有更高的效率。

**$a \% (2^n)$ 等价于 $a \& (2^n - 1)$ **




### tableSizeFor
table数组的大小要求是2的幂次方，如果构造方法中指定的容量并不是2的幂次方
```java
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

### put添加及更新
```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

/**
 * Implements Map.put and related methods
 *
 * @param hash hash for key
 * @param key the key
 * @param value the value to put
 * @param onlyIfAbsent if true, don't change existing value
 * @param evict if false, the table is in creation mode.
 * @return previous value, or null if none
 */
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
	// 使用局部遍历tab而不是类成员，方法栈上访问更快
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // table数组延迟加载，通过resize初始化
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // hash算法定位桶的索引位置
    if ((p = tab[i = (n - 1) & hash]) == null)
    	// 桶位位空时，直接创建一个新的Node节点，将其放入桶位
        tab[i] = newNode(hash, key, value, null);
    // 桶中已经存在单链表或红黑树
    else {
        Node<K,V> e; K k;
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            // 如果键hash值相等，且键也相等
            e = p;
        // 插入到红黑树中
        else if (p instanceof TreeNode)
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        // 插入到单链表中
        else {
        	// 遍历链表，并统计该桶中的Node数量
            for (int binCount = 0; ; ++binCount) {
            	// 该key不在链表中，插入末尾
                if ((e = p.next) == null) {
                    p.next = newNode(hash, key, value, null);
                    // 桶中的节点数大于阈值，转换为红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    break;
                }
                // 该key已经在链表中
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;
                p = e;
            }
        }

        // onlyIfAbsent参数主要决定是否执行替换，当键存在时。
        if (e != null) { // existing mapping for key
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)
                e.value = value;
            // 调用回调函数，HashMap中没有实现具体行为
            afterNodeAccess(e);
            return oldValue;
        }
    }
    ++modCount;
    if (++size > threshold)
    	// 超过阈值，扩容
        resize();
    // 插入完成后的回调，HashMap中没有实现具体行为
    afterNodeInsertion(evict);
    return null;
}
```

### 扩容
1. 首先是调整capacity和threshold，非边际情况下，都变为原来的两倍。
2. 将oldTable的元素逐个迁移到newTable里。
3. 迁移优化
jdk1.7中需要重新计算每个元素的hash值和元素在数组中的位置。但是在jdk1.8中做了优化，因为capacity总是扩展为原来的两倍，只需要看看原来的hash值新增的那个bit是1还是0就可以，是0的话索引没变，是1的话索引变成“原索引+oldCap”。


```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
    	// 超过最大值就不再扩充了
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 没有超过最大值，就扩充为原来的2倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    // oldCap <= 0，table空间尚未分配，初始化分配空间
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {               // zero initial threshold signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 计算新的resize上限
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
    	// 分配新表空间
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    // 原来的表中有内容，表明这是一次扩容，把每个bucket都移动到新的桶中
    if (oldTab != null) {
    	// 遍历所有桶
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                // 该桶中只有一个节点，直接散列到新的位置
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                // 该桶中是一颗红黑树，通过红黑树的split方法处理
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
               	// 单向链表，重新散列，保证原来的顺序
                // 因为容量加倍，散列时（取模运算得到散列的桶索引位置）使用的位数扩展了一位
                // 该链表中的Node散列后可能有两个位置，通过新扩展位为0或1区分
                else { // preserve order
                	// 原链表分为两个链表，一高一低，通过新扩展位来确定
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        // 原索引，新扩展位为0(oldCap为2^k)，低链表
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        // 原索引 + oldCap，高链表
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    // 原索引（低链表）到桶里
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    // 原索引+oldCap 放到桶里（高链表的位置相对于低链表的偏移为oldCap）
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```

### 查找

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // 根节点就命中
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
        	// 在红黑树中查找
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            // 在单链表中查找
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}

public boolean containsKey(Object key) {
    return getNode(hash(key), key) != null;
}

public boolean containsValue(Object value) {
    Node<K,V>[] tab; V v;
    if ((tab = table) != null && size > 0) {
    	// 单链表方式进行遍历（包括红黑树）
        for (int i = 0; i < tab.length; ++i) {
            for (Node<K,V> e = tab[i]; e != null; e = e.next) {
                if ((v = e.value) == value ||
                    (value != null && value.equals(v)))
                    return true;
            }
        }
    }
    return false;
}
```

### 删除
```java
public V remove(Object key) {
    Node<K,V> e;
    return (e = removeNode(hash(key), key, null, false, true)) == null ?
        null : e.value;
}

/**
 * Implements Map.remove and related methods
 *
 * @param hash hash for key
 * @param key the key
 * @param value the value to match if matchValue, else ignored
 * @param matchValue if true only remove if value is equal
 * @param movable if false do not move other nodes while removing
 * @return the node, or null if none
 */
final Node<K,V> removeNode(int hash, Object key, Object value,
                           boolean matchValue, boolean movable) {
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null) {
        // p指向桶中的第一个节点Node
        // node指向目标节点
        Node<K,V> node = null, e; K k; V v;
        // 桶中第一个Node节点命中
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            node = p;
        else if ((e = p.next) != null) {
        	// 红黑树中查找目标节点
            if (p instanceof TreeNode)
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            else {
            	// 单链表中查找目标节点
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                         (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        // 找到目标节点并符合移除条件
        if (node != null && (!matchValue || (v = node.value) == value ||
                             (value != null && value.equals(v)))) {
			// 从红黑树中移除
            if (node instanceof TreeNode)
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
            // 移除单链表的第一个节点
            else if (node == p)
                tab[index] = node.next;
            // 不是单链表的第一个节点
            else
                p.next = node.next;
            ++modCount;
            --size;
            // 删除回调
            afterNodeRemoval(node);
            return node;
        }
    }
    return null;
}

public void clear() {
    Node<K,V>[] tab;
    modCount++;
    if ((tab = table) != null && size > 0) {
        size = 0;
        for (int i = 0; i < tab.length; ++i)
            tab[i] = null;
    }
}
```

### 遍历
HashMap提供了三种方式遍历其中的Entry、Key和Value，分别是`Set<Map.Entry<K, V>> entrySet()`，`Set<K> keySet()`和`Collection<V> values()`。
#### 迭代器
```java
abstract class HashIterator {
    Node<K,V> next;        // next entry to return
    Node<K,V> current;     // current entry
    int expectedModCount;  // for fast-fail
    int index;             // current slot

    HashIterator() {
    	// 保存迭代器创建时的modCount
        expectedModCount = modCount;
        Node<K,V>[] t = table;
        current = next = null;
        index = 0;
        // 找到第一个有效的槽
        if (t != null && size > 0) { // advance to first entry
            do {} while (index < t.length && (next = t[index++]) == null);
        }
    }

    public final boolean hasNext() {
        return next != null;
    }

    final Node<K,V> nextNode() {
        Node<K,V>[] t;
        Node<K,V> e = next;
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        if (e == null)
            throw new NoSuchElementException();
        // 单链表方式遍历
        if ((next = (current = e).next) == null && (t = table) != null) {
        	// 找到下一个有效的槽
            do {} while (index < t.length && (next = t[index++]) == null);
        }
        return e;
    }

    public final void remove() {
        Node<K,V> p = current;
        if (p == null)
            throw new IllegalStateException();
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        current = null;
        K key = p.key;
        removeNode(hash(key), key, null, false, false);
        expectedModCount = modCount;
    }
}

final class KeyIterator extends HashIterator
    implements Iterator<K> {
    public final K next() { return nextNode().key; }
}

final class ValueIterator extends HashIterator
    implements Iterator<V> {
    public final V next() { return nextNode().value; }
}

final class EntryIterator extends HashIterator
    implements Iterator<Map.Entry<K,V>> {
    public final Map.Entry<K,V> next() { return nextNode(); }
}
```
#### 遍历方法
```java
public Set<Map.Entry<K,V>> entrySet() {
    Set<Map.Entry<K,V>> es;
    return (es = entrySet) == null ? (entrySet = new EntrySet()) : es;
}

public Set<K> keySet() {
    Set<K> ks = keySet;
    if (ks == null) {
        ks = new KeySet();
        keySet = ks;
    }
    return ks;
}

public Collection<V> values() {
    Collection<V> vs = values;
    if (vs == null) {
        vs = new Values();
        values = vs;
    }
    return vs;
}

final class EntrySet extends AbstractSet<Map.Entry<K,V>> {
    public final int size()                 { return size; }
    public final void clear()               { HashMap.this.clear(); }
    // 返回一个迭代器
    public final Iterator<Map.Entry<K,V>> iterator() {
        return new EntryIterator();
    }
    public final boolean contains(Object o) {
    	// 这里会导致包含key为null的Map，在此返回false
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry<?,?> e = (Map.Entry<?,?>) o;
        Object key = e.getKey();
        Node<K,V> candidate = getNode(hash(key), key);
        return candidate != null && candidate.equals(e);
    }
    public final boolean remove(Object o) {
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>) o;
            Object key = e.getKey();
            Object value = e.getValue();
            return removeNode(hash(key), key, value, true, true) != null;
        }
        return false;
    }
    public final Spliterator<Map.Entry<K,V>> spliterator() {
        return new EntrySpliterator<>(HashMap.this, 0, -1, 0, 0);
    }
    public final void forEach(Consumer<? super Map.Entry<K,V>> action) {
        Node<K,V>[] tab;
        if (action == null)
            throw new NullPointerException();
        if (size > 0 && (tab = table) != null) {
            int mc = modCount;
            for (int i = 0; i < tab.length; ++i) {
                for (Node<K,V> e = tab[i]; e != null; e = e.next)
                    action.accept(e);
            }
            if (modCount != mc)
                throw new ConcurrentModificationException();
        }
    }
}
```

* * *
感谢：
http://blog.jrwang.me/2016/java-collections-hashmap/
http://liujiacai.net/blog/2015/09/03/java-hashmap/
http://tech.meituan.com/java-hashmap.html
http://www.importnew.com/20321.html