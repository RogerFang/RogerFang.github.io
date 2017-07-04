---
title: 'Java集合框架(7): LinkedHashMap源码分析'
date: 2016-12-29 15:31:50
categories:
- Java集合框架
tags:
- Map
- LinkedHashMap
---

```java
public class LinkedHashMap<K,V>
    extends HashMap<K,V>
    implements Map<K,V>
```

# 简介
`LinkedHashMap`是`HashMap`的子类，也是对`Map`接口的一种基于链表和哈希表的实现，不过扩展了HashMap**增加了双向链表的实现**。
> 相较于HashMap的迭代器中混乱的访问顺序，LinkedHashMap可以提供可以预测的迭代访问，即按照**插入顺序**(insertion-order)或**访问序**(access-order)来对哈希表中的元素进行迭代。

* **插入序**(insertion-order)：就是Entry被添加到Map中的顺序。
* **访问序**(access-order)：是对所有Entry按照**最近访问**(least-recently)到**最远访问**(most-recently)进行排序，读写都会影响到访问顺序，但是对迭代器(`entrySet()`、`keySet()`、`values()`)的访问不会影响到访问顺序。访问顺序使得可以通过LinkedHashMap来实现一个**LRU**(least-recently-used)Cache。

LinkedHashMap 继承自 HashMap，并在其基本结构上增加了双向链表的实现，因而 LinkedHashMap 在内存占用上要比 HashMap 高出许多。
LinkedHashMap 仍然沿用了 HashMap 中基于桶数组、桶内单链表和红黑树结构的哈希表，**在哈希计算、定位、扩容等方面都和 HashMAp 是一致的。LinkedHashMap 同样支持为 null 的键和值。**

# 源码分析
## 属性
```java
/**
 * The head (eldest) of the doubly linked list.
 */
transient LinkedHashMap.Entry<K,V> head;

/**
 * The tail (youngest) of the doubly linked list.
 */
transient LinkedHashMap.Entry<K,V> tail;

/**
 * 迭代顺序，true使用最近被访问的顺序，false为插入顺序
 * The iteration ordering method for this linked hash map:
 * true for access-order,
 * false for insertion-order.
 *
 * @serial
 */
final boolean accessOrder;
```

## 构造方法
LinkedHashMap的构造函数总是在第一行调用父类构造函数，accessOrder默认为false（即默认按照插入顺序访问）。
```java
public LinkedHashMap(int initialCapacity, float loadFactor) {
    super(initialCapacity, loadFactor);
    accessOrder = false;
}

public LinkedHashMap(int initialCapacity) {
    super(initialCapacity);
    accessOrder = false;
}

public LinkedHashMap() {
    super();
    accessOrder = false;
}

public LinkedHashMap(Map<? extends K, ? extends V> m) {
    super();
    accessOrder = false;
    putMapEntries(m, false);
}

public LinkedHashMap(int initialCapacity,
                     float loadFactor,
                     boolean accessOrder) {
    super(initialCapacity, loadFactor);
    this.accessOrder = accessOrder;
}
```

## 节点
`LinkedHashMap.Entry`继承自`HashMap.Node`，而`HashMap.TreeNode`又继承了`LinkedHashMap.Entry`。
LinkedHashMap**只会对双向链表的关系进行管理，单向链表的关系仍由其父类进行管理。**
```java
static class Entry<K,V> extends HashMap.Node<K,V> {
	// 仍保留父类HashMap的节点Node中存在的next引用，可以将每个桶的元素都当作一个单链表来看待

	// 实现双向链表，增加了before和after
    Entry<K,V> before, after;
    Entry(int hash, K key, V value, Node<K,V> next) {
        super(hash, key, value, next);
    }
}
```

## 方法
### 创建节点
重写了父类HashMap的newNode和newTreeNode方法，并将创建的新节点插入到双链表末尾。
```java
Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
    LinkedHashMap.Entry<K,V> p =
        new LinkedHashMap.Entry<K,V>(hash, key, value, e);
    linkNodeLast(p);
    return p;
}

TreeNode<K,V> newTreeNode(int hash, K key, V value, Node<K,V> next) {
    TreeNode<K,V> p = new TreeNode<K,V>(hash, key, value, next);
    linkNodeLast(p);
    return p;
}
```

### 链接变化
```java
// link at the end of list
// 将新节点p链接到双向链表的末尾
private void linkNodeLast(LinkedHashMap.Entry<K,V> p) {
    LinkedHashMap.Entry<K,V> last = tail;
    tail = p;
    if (last == null)
    	// 为空，则为头节点
        head = p;
    else {
    	// 修改尾节点
        p.before = last;
        last.after = p;
    }
}

// apply src's links to dst
// 用dst替换src在双向链表中的位置
private void transferLinks(LinkedHashMap.Entry<K,V> src,
                           LinkedHashMap.Entry<K,V> dst) {
	// 修改dst的前驱和后继
    LinkedHashMap.Entry<K,V> b = dst.before = src.before;
    LinkedHashMap.Entry<K,V> a = dst.after = src.after;
    // 将双向链表中原来指向src的链接改为指向dst
    if (b == null)
        head = dst;
    else
        b.after = dst;
    if (a == null)
        tail = dst;
    else
        a.before = dst;
}
```

### 节点变化
```java
/**
 * 将节点由TreeNode转换为普通节点Node
 */
Node<K,V> replacementNode(Node<K,V> p, Node<K,V> next) {
    LinkedHashMap.Entry<K,V> q = (LinkedHashMap.Entry<K,V>)p;
    LinkedHashMap.Entry<K,V> t =
        new LinkedHashMap.Entry<K,V>(q.hash, q.key, q.value, next);
    transferLinks(q, t);
    return t;
}

/**
 * 将节点由普通节点转换为TreeNode
 */
TreeNode<K,V> replacementTreeNode(Node<K,V> p, Node<K,V> next) {
    LinkedHashMap.Entry<K,V> q = (LinkedHashMap.Entry<K,V>)p;
    TreeNode<K,V> t = new TreeNode<K,V>(q.hash, q.key, q.value, next);
    transferLinks(q, t);
    return t;
}
```

### 维护插入序和访问序
在新建节点的时候，都是将新建节点链接到双向链表的末尾。从双向链表的尾部向头部遍历就可以保证**插入顺序**了。

在插入节点、删除节点和访问节点后会调用相应的回调函数（在HashMap中是空实现），LinkedHashMap通过实现这些回调函数实现了访问顺序的维护。
```java
/**
 * 插入节点的回调函数
 */
void afterNodeInsertion(boolean evict) { // possibly remove eldest
	// first头元素
    // 插入序：代表最先插入的元素
    // 访问序：代表最远被访问的元素
    LinkedHashMap.Entry<K,V> first;
    // removeEldestEntry始终返回false，即不删除最老的元素
    // 如果需要一个容量固定的cache，可调整removeEldestEntry的实现
    if (evict && (first = head) != null && removeEldestEntry(first)) {
    	// evict为true，表示非构造方法中调用
        K key = first.key;
        removeNode(hash(key), key, null, false, true);
    }
}

/**
 * 移除节点的回调函数
 */
void afterNodeRemoval(Node<K,V> e) { // unlink
	// 移除节点，调整链表中的链接关系
    LinkedHashMap.Entry<K,V> p =
        (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
    p.before = p.after = null;
    if (b == null)
        head = a;
    else
        b.after = a;
    if (a == null)
        tail = b;
    else
        a.before = b;
}

/**
 * 访问节点的回调函数
 */
void afterNodeAccess(Node<K,V> e) { // move node to last
    LinkedHashMap.Entry<K,V> last;
    // 如果是访问序，且当前节点不是尾节点，则将该节点设为尾节点
    if (accessOrder && (last = tail) != e) {
    	// p为当前节点
        LinkedHashMap.Entry<K,V> p =
            (LinkedHashMap.Entry<K,V>)e, b = p.before, a = p.after;
        p.after = null;
        // p是头节点
        if (b == null)
            head = a;
        else
            b.after = a;
        if (a != null)
            a.before = b;
        else
            last = b;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
        tail = p;
        ++modCount;
    }
}
```

### 遍历
LinkedHashMap的所有节点都在一个双向链表中，因此可以通过双向链表来遍历所有的Entry。
#### 迭代器
```java
abstract class LinkedHashIterator {
    LinkedHashMap.Entry<K,V> next;
    LinkedHashMap.Entry<K,V> current;
    int expectedModCount;

    LinkedHashIterator() {
        next = head;
        expectedModCount = modCount;
        current = null;
    }

    public final boolean hasNext() {
        return next != null;
    }

    final LinkedHashMap.Entry<K,V> nextNode() {
        LinkedHashMap.Entry<K,V> e = next;
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
        if (e == null)
            throw new NoSuchElementException();
        current = e;
        next = e.after;
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

final class LinkedKeyIterator extends LinkedHashIterator
    implements Iterator<K> {
    public final K next() { return nextNode().getKey(); }
}

final class LinkedValueIterator extends LinkedHashIterator
    implements Iterator<V> {
    public final V next() { return nextNode().value; }
}

final class LinkedEntryIterator extends LinkedHashIterator
    implements Iterator<Map.Entry<K,V>> {
    public final Map.Entry<K,V> next() { return nextNode(); }
}
```

#### 遍历方法
```java
public Set<K> keySet() {
    Set<K> ks = keySet;
    if (ks == null) {
        ks = new LinkedKeySet();
        keySet = ks;
    }
    return ks;
}

public Collection<V> values() {
    Collection<V> vs = values;
    if (vs == null) {
        vs = new LinkedValues();
        values = vs;
    }
    return vs;
}

public Set<Map.Entry<K,V>> entrySet() {
    Set<Map.Entry<K,V>> es;
    return (es = entrySet) == null ? (entrySet = new LinkedEntrySet()) : es;
}

final class LinkedEntrySet extends AbstractSet<Map.Entry<K,V>> {
    public final int size()                 { return size; }
    public final void clear()               { LinkedHashMap.this.clear(); }
    public final Iterator<Map.Entry<K,V>> iterator() {
        return new LinkedEntryIterator();
    }
    public final boolean contains(Object o) {
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
        return Spliterators.spliterator(this, Spliterator.SIZED |
                                        Spliterator.ORDERED |
                                        Spliterator.DISTINCT);
    }
    public final void forEach(Consumer<? super Map.Entry<K,V>> action) {
        if (action == null)
            throw new NullPointerException();
        int mc = modCount;
        for (LinkedHashMap.Entry<K,V> e = head; e != null; e = e.after)
            action.accept(e);
        if (modCount != mc)
            throw new ConcurrentModificationException();
    }
}
```


* * *

感谢：
http://blog.jrwang.me/2016/java-collections-linkedhashmap/