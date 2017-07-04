---
title: 'Java集合框架(8): TreeMap源码分析'
date: 2016-12-29 18:03:26
categories:
- Java集合框架
tags:
- Map
- TreeMap
---

```java
public class TreeMap<K,V>
    extends AbstractMap<K,V>
    implements NavigableMap<K,V>, Cloneable, java.io.Serializable
```

# 简介
`TreeMap`是一种基于**红黑树**实现的key-value结构，非线程安全的。
如果没有使用自定义的Comparator对Key为null进行相应的处理，则不支持Key为null。

> 关于红黑树的介绍可以参见：[数据结构与算法(9): 树形结构- 红黑树](https://rogerfang.github.io/2016/10/29/%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84%E4%B8%8E%E7%AE%97%E6%B3%95-9-%E6%A0%91%E5%BD%A2%E7%BB%93%E6%9E%84-%E7%BA%A2%E9%BB%91%E6%A0%91/)

在使用集合视图在`HashMap`中迭代时，是不能保证迭代顺序的；`LinkedHashMap`使用了双向链表，保证按照插入顺序或访问顺序进行迭代。`TreeMap`支持**按键**进行排序，可以由传入的比较器（Comparator）来控制（当然，自己也可以实现Comparator利用value进行排序）。

> `SortedMap`对排序进行了相关说明：A Map that further provides a total ordering on its **keys**.


## SortedMap
`SortedMap`接口扩展自`Map`接口，**对该接口的实现要保证所有的Key是完全有序的，这个顺序一般指key的自然序**（实现`Comparable`接口）或在创建`SortedMap`时指定一个比较器（`Comparator`），当使用集合的视图（Collection View，由entrySet、keySet和values方法提供）来迭代时就可以按序访问其中的元素。
> 插入`SortedMap`中的所有Key的类都必须实现`Comparable`接口（或者可以作为指定的`Comparator`的参数）。


`SortedMap`中的Key的顺序必须和equals保持一致，即`k1.compareTo(k2) == 0`(or `comparator.compare(k1, k2) == 0`)和`k1.equals(k2)`要有相同的布尔值。
> 这是因为 Map 接口的定义中，比较 Key 是通过 equals 方法，而在 SortedMap 中比较 Key 则是通过 compareTo (or compare) 方法。如果不一致的，就破坏了 Map 接口的约定。

## NavigableMap
`NavigableMap`时JDK 1.6之后新增的接口，扩展了`SortedMap`接口，提供了一些导航方法来返回最接近搜索目标的匹配结果。
* `lowerEntry(K key)` (or `lowerKey(K key)`)，**小于**给定 Key 的 Entry (or Key)
* `floorEntry(K key)` (or `floorKey(K key)`)，**小于等于**给定 Key 的 Entry (or Key)
* `higherEntry(K key)` (or `higherKey(K key)`)，**大于**给定 Key 的 Entry (or Key)
* `ceilingEntry(K key)` (or `ceilingKey(K key)`)，**大于等于**给定 Key 的 Entry (or Key)

`NavigableMap` 可以按照 Key 的升序或降序进行访问和遍历。 `descendingMap()` 和 `descendingKeySet()` 则会获取和原来的顺序相反的集合，集合中的元素则是同样的引用，在该视图上的修改会影响到原始的数据。

# 源码分析
## 属性
```java
	// 比较器，没有指定的默认使用Key的自然序
    private final Comparator<? super K> comparator;
	// 红黑树的根节点
    private transient Entry<K,V> root;
	// 树种节点的数量
    private transient int size = 0;
	// 结构化修改的次数
    private transient int modCount = 0;

    private transient EntrySet entrySet;
    private transient KeySet<K> navigableKeySet;
    private transient NavigableMap<K,V> descendingMap;
```

## 构造方法

## 红黑树节点
```java
static final class Entry<K,V> implements Map.Entry<K,V> {
    K key;
    V value;
    Entry<K,V> left;
    Entry<K,V> right;
    Entry<K,V> parent;
    boolean color = BLACK;

    /**
     * Make a new cell with given key, value, and parent, and with
     * {@code null} child links, and BLACK color.
     */
    Entry(K key, V value, Entry<K,V> parent) {
        this.key = key;
        this.value = value;
        this.parent = parent;
    }

    /**
     * Returns the key.
     *
     * @return the key
     */
    public K getKey() {
        return key;
    }

    /**
     * Returns the value associated with the key.
     *
     * @return the value associated with the key
     */
    public V getValue() {
        return value;
    }

    /**
     * Replaces the value currently associated with the key with the given
     * value.
     *
     * @return the value associated with the key before this method was
     *         called
     */
    public V setValue(V value) {
        V oldValue = this.value;
        this.value = value;
        return oldValue;
    }

    public boolean equals(Object o) {
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry<?,?> e = (Map.Entry<?,?>)o;

        return valEquals(key,e.getKey()) && valEquals(value,e.getValue());
    }

    /**
     * 哈希值的计算，Key和Value的哈希值进行位异或运算
     */
    public int hashCode() {
        int keyHash = (key==null ? 0 : key.hashCode());
        int valueHash = (value==null ? 0 : value.hashCode());
        return keyHash ^ valueHash;
    }

    public String toString() {
        return key + "=" + value;
    }
}
```

## 方法
### put添加及更新
为了维持红黑树的有序，添加及更新的代价较高，复杂度为$O(\log{N})$。
```java
public V put(K key, V value) {
    Entry<K,V> t = root;
    if (t == null) {
        compare(key, key); // type (and possibly null) check

        root = new Entry<>(key, value, null);
        size = 1;
        modCount++;
        return null;
    }
    int cmp;
    Entry<K,V> parent;
    // split comparator and comparable paths
    Comparator<? super K> cpr = comparator;
    // 比较器，使用定制的排序方法
    if (cpr != null) {
        do {
            parent = t;
            cmp = cpr.compare(key, t.key);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
                return t.setValue(value);
        } while (t != null);
    }
    // 比较器为空，Key必须实现Comparable接口
    else {
        if (key == null)
            throw new NullPointerException();
        @SuppressWarnings("unchecked")
            Comparable<? super K> k = (Comparable<? super K>) key;
        do {
            parent = t;
            cmp = k.compareTo(t.key);
            if (cmp < 0)
                t = t.left;
            else if (cmp > 0)
                t = t.right;
            else
            	// Key存在，更新Value
                return t.setValue(value);
        } while (t != null);
    }
    // Key不存在，创建新节点，插入二叉树
    Entry<K,V> e = new Entry<>(key, value, parent);
    if (cmp < 0)
        parent.left = e;
    else
        parent.right = e;
	// 插入后修复红黑树
    fixAfterInsertion(e);
    size++;
    modCount++;
    return null;
}
```

### 删除
红黑树的删除较为复杂
```java
public V remove(Object key) {
	// 先查找该节点
    Entry<K,V> p = getEntry(key);
    if (p == null)
        return null;

    V oldValue = p.value;
    // 删除该节点
    deleteEntry(p);
    return oldValue;
}

/**
 * Delete node p, and then rebalance the tree.
 */
private void deleteEntry(Entry<K,V> p) {
    modCount++;
    size--;

    // If strictly internal, copy successor's element to p and then make p
    // point to successor.
    // 被删除节点的左右子树都不为空
    if (p.left != null && p.right != null) {
    	// 用后继节点代替当前节点
        Entry<K,V> s = successor(p);
        p.key = s.key;
        p.value = s.value;
        p = s;
    } // p has 2 children

    // Start fixup at replacement node, if it exists.
    // 左子节点存在，则replacement为左子节点，否则为右子节点
    Entry<K,V> replacement = (p.left != null ? p.left : p.right);

	// 至少有一个子节点存在
    if (replacement != null) {
        // Link replacement to parent
        replacement.parent = p.parent;
        // p是根节点
        if (p.parent == null)
            root = replacement;
        // p是父节点的左子节点
        else if (p == p.parent.left)
            p.parent.left  = replacement;
        // p是父节点的右子节点
        else
            p.parent.right = replacement;

        // Null out links so they are OK to use by fixAfterDeletion.
        p.left = p.right = p.parent = null;

        // Fix replacement
        if (p.color == BLACK)
            fixAfterDeletion(replacement);
    } else if (p.parent == null) { // return if we are the only node.
    	// 没有父节点，则该节点是树中的唯一节点
        root = null;
    } else { //  No children. Use self as phantom replacement and unlink.
    	// 没有子节点
        if (p.color == BLACK)
            fixAfterDeletion(p);

        if (p.parent != null) {
            if (p == p.parent.left)
                p.parent.left = null;
            else if (p == p.parent.right)
                p.parent.right = null;
            p.parent = null;
        }
    }
}
```

### 查找
红黑树的查找复杂度为$O(\log{N})$。
```java
public V get(Object key) {
    Entry<K,V> p = getEntry(key);
    return (p==null ? null : p.value);
}

final Entry<K,V> getEntry(Object key) {
    // Offload comparator-based version for sake of performance
    // 使用定制的比较器
    if (comparator != null)
        return getEntryUsingComparator(key);
    if (key == null)
        throw new NullPointerException();
    @SuppressWarnings("unchecked")
        Comparable<? super K> k = (Comparable<? super K>) key;
    Entry<K,V> p = root;
    while (p != null) {
        int cmp = k.compareTo(p.key);
        if (cmp < 0)
            p = p.left;
        else if (cmp > 0)
            p = p.right;
        else
            return p;
    }
    return null;
}

final Entry<K,V> getEntryUsingComparator(Object key) {
    @SuppressWarnings("unchecked")
        K k = (K) key;
    Comparator<? super K> cpr = comparator;
    if (cpr != null) {
        Entry<K,V> p = root;
        while (p != null) {
            int cmp = cpr.compare(k, p.key);
            if (cmp < 0)
                p = p.left;
            else if (cmp > 0)
                p = p.right;
            else
                return p;
        }
    }
    return null;
}
```

### 包含
```java
public boolean containsKey(Object key) {
    return getEntry(key) != null;
}

public boolean containsValue(Object value) {
    for (Entry<K,V> e = getFirstEntry(); e != null; e = successor(e))
        if (valEquals(value, e.value))
            return true;
    return false;
}
```

### 遍历
```java
/**
 * TreeMap中所有迭代器的基础
 */
abstract class PrivateEntryIterator<T> implements Iterator<T> {
    Entry<K,V> next;
    Entry<K,V> lastReturned;
    int expectedModCount;

    PrivateEntryIterator(Entry<K,V> first) {
        expectedModCount = modCount;
        lastReturned = null;
        next = first;
    }

    public final boolean hasNext() {
        return next != null;
    }

    final Entry<K,V> nextEntry() {
        Entry<K,V> e = next;
        if (e == null)
            throw new NoSuchElementException();
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        // 后继节点
        next = successor(e);
        lastReturned = e;
        return e;
    }

    final Entry<K,V> prevEntry() {
        Entry<K,V> e = next;
        if (e == null)
            throw new NoSuchElementException();
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        // 前驱节点
        next = predecessor(e);
        lastReturned = e;
        return e;
    }

    public void remove() {
        if (lastReturned == null)
            throw new IllegalStateException();
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        // deleted entries are replaced by their successors
        if (lastReturned.left != null && lastReturned.right != null)
            next = lastReturned;
        deleteEntry(lastReturned);
        expectedModCount = modCount;
        lastReturned = null;
    }
}

final class EntryIterator extends PrivateEntryIterator<Map.Entry<K,V>> {
    EntryIterator(Entry<K,V> first) {
        super(first);
    }
    public Map.Entry<K,V> next() {
        return nextEntry();
    }
}

final class ValueIterator extends PrivateEntryIterator<V> {
    ValueIterator(Entry<K,V> first) {
        super(first);
    }
    public V next() {
        return nextEntry().value;
    }
}

final class KeyIterator extends PrivateEntryIterator<K> {
    KeyIterator(Entry<K,V> first) {
        super(first);
    }
    public K next() {
        return nextEntry().key;
    }
}

final class DescendingKeyIterator extends PrivateEntryIterator<K> {
    DescendingKeyIterator(Entry<K,V> first) {
        super(first);
    }
    public K next() {
        return prevEntry().key;
    }
    public void remove() {
        if (lastReturned == null)
            throw new IllegalStateException();
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        deleteEntry(lastReturned);
        lastReturned = null;
        expectedModCount = modCount;
    }
}
```

### 前驱和后继
```java
/**
 * 后继节点
 * Returns the successor of the specified Entry, or null if no such.
 */
static <K,V> TreeMap.Entry<K,V> successor(Entry<K,V> t) {
    if (t == null)
        return null;
    else if (t.right != null) {
    	// 右子树存在，则取右子树的最小节点
        Entry<K,V> p = t.right;
        while (p.left != null)
            p = p.left;
        return p;
    } else {
    	// 右子树不存在
        // 若父节点为null，则该节点是最大节点，无后继，返回null
        // 若当前节点是父节点的左子节点，直接返回父节点
        // 若当前节点是父节点的右子节点，则当前节点是以其父节点为根的子树的最大节点
        Entry<K,V> p = t.parent; //父节点
        Entry<K,V> ch = t; //当前节点
        while (p != null && ch == p.right) {
            ch = p;
            p = p.parent;
        }
        return p;
    }
}

/**
 * 前驱节点
 * Returns the predecessor of the specified Entry, or null if no such.
 */
static <K,V> Entry<K,V> predecessor(Entry<K,V> t) {
    if (t == null)
        return null;
    else if (t.left != null) {
        Entry<K,V> p = t.left;
        while (p.right != null)
            p = p.right;
        return p;
    } else {
        Entry<K,V> p = t.parent;
        Entry<K,V> ch = t;
        while (p != null && ch == p.left) {
            ch = p;
            p = p.parent;
        }
        return p;
    }
}
```


* * *
感谢：
http://blog.jrwang.me/2016/java-collections-treemap/