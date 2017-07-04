---
title: 'JUC集合框架(2): ConcurrentHashMap'
date: 2017-01-16 11:03:06
categories:
- JUC集合框架
tags:
- Map
- ConcurrentHashMap
---

```java
public class ConcurrentHashMap<K,V> extends AbstractMap<K,V>
    implements ConcurrentMap<K,V>, Serializable
```

`ConcurrentHashMap`相当于线程安全的`HashMap`，它继承于`AbstractMap`类，并实现了`ConcurrentMap`接口。

**ConcurrentHashMap不允许key或value为null值**。

# ConcurrentHashMap、HashMap和HashTable
为什么要使用ConcurrentHashMap，而不是HashMap或者HashTable？
1. 线程不安全的HashMap
	在多线程环境下，使用HashMap进行put操作会导致HashMap中的Entry链表形成环形数据结构，从而使得在进行get等操作时引起死循环。

2. 效率低下的HashTable
	HashTable使用synchronized来保证线程安全，但是在线程竞争激烈的情况下效率低下。因为HashTable会对整个表进行加锁来实现同步，而JDK 1.8中的ConcurrentHashMap则使用更细粒度的锁来实现同步（对表中的某一列进行加锁）。

HashTable 虽然性能上不如 ConcurrentHashMap，但**并不能完全被取代**，两者的迭代器的一致性不同的，HashTable的迭代器是强一致性的，而**ConcurrentHashMap是弱一致的**。 ConcurrentHashMap的get，clear，iterator 都是弱一致性的。

# 原理和结构
## JDK 1.8之前
JDK 1.8 之前的`ConcurrentHashMap`是通过**锁分段**来实现的，只有在同一个分段内才存在竞态关系。由**Segment**数组结构和**HashEntry**数组结构组成的。
一个ConcurrentHashMap里包含一个Segment数组，一个Segment里包含一个HashEntry数组。
![](/images/juc/juc-concurrenthashmap-java7.png)

`Segment`是一个**可重入锁**（ReentrantLock），在ConcurrentHashMap里扮演锁的角色。`HashEntry`用于存储键值，每个HashEntry是一个链表结构的元素。

每个Segment守护者一个HashEntry数组里面的元素，当对HashEntry数组里的数据进行修改时，必须首先获得与它对应的Segment锁。
Segment的结构和`HashMap`的结构类似，是一种数组和链表的结构。

## JDK 1.8
JDK 1.8中，`ConcurrentHashMap`的实现已经抛弃了Segment分段锁机制，大量的利用了volatile，final，CAS等lock-free技术来减少锁竞争对于性能的影响。
**CAS**+**Synchronized**来保证并发更新的安全，底层依然采用**数组+链表+红黑树**的存储结构。但是为了做到并发，又增加了很多辅助的类，例如TreeBin，Traverser等对象内部类。

synchronized关键字锁住 table 中本次所操作对象的**链表头**。

![](/images/juc/juc-concurrenthashmap-java8.png)

现在的实现中并没有用segment，而是延用 hashMap中单桶，定位到链表后，直接上锁，而后对链表就行操作，同样是将锁细化。
> Segment虽保留，但已经简化属性，仅仅是为了兼容旧版本。

# JDK 1.8实现
## 属性
```java
// 桶，大小是2的整数次幂
transient volatile Node<K,V>[] table;
// hash表初始化或扩容时的一个控制位标识量
private transient volatile int sizeCtl;

// 一个过渡的table表，只在扩容的时候会用到
private transient volatile Node<K,V>[] nextTable;
```
### sizeCtl
`sizeCtl`属性是个控制标识位：
* 0，默认初始值
* 负数，表示正在进行初始化或扩容操作控制标识符
	* -1，表示正在初始化。
	* -N，表示有N-1个线程正在进行扩容操作。
* 正数
	* 如果table未初始化，表示table需要初始化的大小；
	* 如果table初始化完成，表示下一次扩容的扩容阈值，默认是table大小的0.75倍。实际容量>=sizeCtl，则扩容。

## 内部类
### Node
Node 是最核心的内部类，包装了key-value键值对。它和HashMap中的定义很相似，但是value和next属性设置为volatile，不允许调用`setValue()`方法直接改变Node的value域，增加了`find()`方法辅助`map.get()`方法。
```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    volatile V val;
    volatile Node<K,V> next;

    Node(int hash, K key, V val, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.val = val;
        this.next = next;
    }

    public final V setValue(V value) {
        throw new UnsupportedOperationException();
    }

    /**
     * Virtualized support for map.get(); overridden in subclasses.
     */
    Node<K,V> find(int h, Object k) {
        Node<K,V> e = this;
        if (k != null) {
            do {
                K ek;
                if (e.hash == h &&
                    ((ek = e.key) == k || (ek != null && k.equals(ek))))
                    return e;
            } while ((e = e.next) != null);
        }
        return null;
    }

    ...
}
```

### TreeNode
当链表长度过长的时候，会转换为TreeNode。
但是与HashMap不相同的是，它并不是直接转换为红黑树，而是把这些结点包装成`TreeNode`放在`TreeBin`对象中，由`TreeBin`完成对红黑树的包装。

而且`TreeNode`在ConcurrentHashMap集成自`Node`类，而并非HashMap中的集成自`LinkedHashMap.Entry<K,V>`类，也就是说TreeNode带有next指针，这样做的目的是方便基于TreeBin的访问。
```java
/**
 * Nodes for use in TreeBins
 */
static final class TreeNode<K,V> extends Node<K,V> {
    TreeNode<K,V> parent;  // red-black tree links
    TreeNode<K,V> left;
    TreeNode<K,V> right;
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;

    TreeNode(int hash, K key, V val, Node<K,V> next,
             TreeNode<K,V> parent) {
        super(hash, key, val, next);
        this.parent = parent;
    }

    Node<K,V> find(int h, Object k) {
        return findTreeNode(h, k, null);
    }

    /**
     * Returns the TreeNode (or null if not found) for the given key
     * starting at given root.
     */
    final TreeNode<K,V> findTreeNode(int h, Object k, Class<?> kc) {
        if (k != null) {
            TreeNode<K,V> p = this;
            do  {
                int ph, dir; K pk; TreeNode<K,V> q;
                TreeNode<K,V> pl = p.left, pr = p.right;
                if ((ph = p.hash) > h)
                    p = pl;
                else if (ph < h)
                    p = pr;
                else if ((pk = p.key) == k || (pk != null && k.equals(pk)))
                    return p;
                else if (pl == null)
                    p = pr;
                else if (pr == null)
                    p = pl;
                else if ((kc != null ||
                          (kc = comparableClassFor(k)) != null) &&
                         (dir = compareComparables(kc, k, pk)) != 0)
                    p = (dir < 0) ? pl : pr;
                else if ((q = pr.findTreeNode(h, k, kc)) != null)
                    return q;
                else
                    p = pl;
            } while (p != null);
        }
        return null;
    }
}
```

### TreeBin
这个类并不负责包装用户的key、value信息，而是包装的很多TreeNode节点。它代替了TreeNode的根节点，也就是说在实际的ConcurrentHashMap“数组”中，存放的是TreeBin对象，而不是TreeNode对象，这是与HashMap的区别。另外这个类还带有了读写锁。
```java
static final class TreeBin<K,V> extends Node<K,V> {
    TreeNode<K,V> root;
    volatile TreeNode<K,V> first;
    volatile Thread waiter;
    volatile int lockState;
    // values for lockState
    static final int WRITER = 1; // set while holding write lock
    static final int WAITER = 2; // set when waiting for write lock
    static final int READER = 4; // increment value for setting read lock

    /**
     * Tie-breaking utility for ordering insertions when equal
     * hashCodes and non-comparable. We don't require a total
     * order, just a consistent insertion rule to maintain
     * equivalence across rebalancings. Tie-breaking further than
     * necessary simplifies testing a bit.
     */
    static int tieBreakOrder(Object a, Object b) {
        int d;
        if (a == null || b == null ||
            (d = a.getClass().getName().
             compareTo(b.getClass().getName())) == 0)
            d = (System.identityHashCode(a) <= System.identityHashCode(b) ?
                 -1 : 1);
        return d;
    }

    /**
     * Creates bin with initial set of nodes headed by b.
     */
    TreeBin(TreeNode<K,V> b) {
        super(TREEBIN, null, null, null);
        this.first = b;
        TreeNode<K,V> r = null;
        for (TreeNode<K,V> x = b, next; x != null; x = next) {
            next = (TreeNode<K,V>)x.next;
            x.left = x.right = null;
            if (r == null) {
                x.parent = null;
                x.red = false;
                r = x;
            }
            else {
                K k = x.key;
                int h = x.hash;
                Class<?> kc = null;
                for (TreeNode<K,V> p = r;;) {
                    int dir, ph;
                    K pk = p.key;
                    if ((ph = p.hash) > h)
                        dir = -1;
                    else if (ph < h)
                        dir = 1;
                    else if ((kc == null &&
                              (kc = comparableClassFor(k)) == null) ||
                             (dir = compareComparables(kc, k, pk)) == 0)
                        dir = tieBreakOrder(k, pk);
                        TreeNode<K,V> xp = p;
                    if ((p = (dir <= 0) ? p.left : p.right) == null) {
                        x.parent = xp;
                        if (dir <= 0)
                            xp.left = x;
                        else
                            xp.right = x;
                        r = balanceInsertion(r, x);
                        break;
                    }
                }
            }
        }
        this.root = r;
        assert checkInvariants(root);
    }

    ...
}
```

### ForwardingNode
并不是我们传统的包含key-value的节点，只是一个标志节点，hash值为-1，并且指向nextTable，提供find方法而已。

只有table发生扩容的时候，ForwardingNode才会发挥作用，作为一个占位符放在table中表示当前节点为null或则已经被移动。
```java
/**
 * A node inserted at head of bins during transfer operations.
 */
static final class ForwardingNode<K,V> extends Node<K,V> {
    final Node<K,V>[] nextTable;
    ForwardingNode(Node<K,V>[] tab) {
        super(MOVED, null, null, null);
        this.nextTable = tab;
    }

    Node<K,V> find(int h, Object k) {
        // loop to avoid arbitrarily deep recursion on forwarding nodes
        outer: for (Node<K,V>[] tab = nextTable;;) {
            Node<K,V> e; int n;
            if (k == null || tab == null || (n = tab.length) == 0 ||
                (e = tabAt(tab, (n - 1) & h)) == null)
                return null;
            for (;;) {
                int eh; K ek;
                if ((eh = e.hash) == h &&
                    ((ek = e.key) == k || (ek != null && k.equals(ek))))
                    return e;
                if (eh < 0) {
                    if (e instanceof ForwardingNode) {
                        tab = ((ForwardingNode<K,V>)e).nextTable;
                        continue outer;
                    }
                    else
                        return e.find(h, k);
                }
                if ((e = e.next) == null)
                    return null;
            }
        }
    }
}
```

## Unsafe和CAS
在ConcurrentHashMap中，随处可以看到U, 大量使用了U.compareAndSwapXXX的方法，这个方法是**利用一个CAS算法实现无锁化的修改值的操作**，它可以大大降低锁带来的性能消耗。

算法的基本思想就是：不断地去比较当前内存中的变量值与你指定的一个变量值是否相等，如果相等，则接受你指定的修改的值，否则拒绝你的操作。因为当前线程中的值已经不是最新的值，你的修改很可能会覆盖掉其他线程修改的结果。
### unsafe静态块
```java
// Unsafe mechanics
private static final sun.misc.Unsafe U;
private static final long SIZECTL;
private static final long TRANSFERINDEX;
private static final long BASECOUNT;
private static final long CELLSBUSY;
private static final long CELLVALUE;
private static final long ABASE;
private static final int ASHIFT;

static {
    try {
        U = sun.misc.Unsafe.getUnsafe();
        Class<?> k = ConcurrentHashMap.class;
        SIZECTL = U.objectFieldOffset
            (k.getDeclaredField("sizeCtl"));
        TRANSFERINDEX = U.objectFieldOffset
            (k.getDeclaredField("transferIndex"));
        BASECOUNT = U.objectFieldOffset
            (k.getDeclaredField("baseCount"));
        CELLSBUSY = U.objectFieldOffset
            (k.getDeclaredField("cellsBusy"));
        Class<?> ck = CounterCell.class;
        CELLVALUE = U.objectFieldOffset
            (ck.getDeclaredField("value"));
        Class<?> ak = Node[].class;
        ABASE = U.arrayBaseOffset(ak);
        int scale = U.arrayIndexScale(ak);
        if ((scale & (scale - 1)) != 0)
            throw new Error("data type scale not a power of two");
        ASHIFT = 31 - Integer.numberOfLeadingZeros(scale);
    } catch (Exception e) {
        throw new Error(e);
    }
}
```

### 三个原子操作
ConcurrentHashMap定义了三个原子操作，用于对指定位置的节点进行操作。正是这些原子操作保证了ConcurrentHashMap的线程安全。
```java
/**
 * 获得位置i上的Node节点
 */
static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) {
    return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
}

/**
 * 利用CAS设置位置i上的Node节点。
 */
static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                    Node<K,V> c, Node<K,V> v) {
    return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
}

/**
 * volatile写，设置节点位置的值。在加锁的情况下使用
 */
static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v) {
    U.putObjectVolatile(tab, ((long)i << ASHIFT) + ABASE, v);
}
```

## 构造方法
`ConcurrentHashMap`的构造方法仅仅是设置了一些参数而已。

而整个table的初始化是在向ConcurrentHashMap中**插入元素**的时候发生的。如调用`put()`、`computeIfAbsent()`、`compute()`、`merge()`等方法的时候，调用时机是检查`table==null`。

```java
public ConcurrentHashMap() {
}

public ConcurrentHashMap(int initialCapacity) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException();
    int cap = ((initialCapacity >= (MAXIMUM_CAPACITY >>> 1)) ?
               MAXIMUM_CAPACITY :
               tableSizeFor(initialCapacity + (initialCapacity >>> 1) + 1));
    this.sizeCtl = cap;
}

public ConcurrentHashMap(Map<? extends K, ? extends V> m) {
    this.sizeCtl = DEFAULT_CAPACITY;
    putAll(m);
}

public ConcurrentHashMap(int initialCapacity, float loadFactor) {
    this(initialCapacity, loadFactor, 1);
}

public ConcurrentHashMap(int initialCapacity,
                         float loadFactor, int concurrencyLevel) {
    if (!(loadFactor > 0.0f) || initialCapacity < 0 || concurrencyLevel <= 0)
        throw new IllegalArgumentException();
    if (initialCapacity < concurrencyLevel)   // Use at least as many bins
        initialCapacity = concurrencyLevel;   // as estimated threads
    long size = (long)(1.0 + (long)initialCapacity / loadFactor);
    int cap = (size >= (long)MAXIMUM_CAPACITY) ?
        MAXIMUM_CAPACITY : tableSizeFor((int)size);
    this.sizeCtl = cap;
}
```

## 初始化initTable
初始化方法主要应用了关键属性`sizeCtl`，如果这个值〈0，表示其他线程正在进行初始化，就放弃这个操作。

在这也可以看出ConcurrentHashMap的**初始化只能由一个线程完成**。如果获得了初始化权限，就用CAS方法将sizeCtl置为-1，防止其他线程进入。初始化数组后，将sizeCtl的值改为0.75*n。
```java
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {
        if ((sc = sizeCtl) < 0)
        	// 有其他线程正在进行初始化操作，把线程挂起。
            Thread.yield(); // lost initialization race; just spin
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) { //利用CAS方法把sizectl的值置为-1，表示正在进行初始化。
            try {
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    sc = n - (n >>> 2); //相当于0.75*n 设置一个扩容的阈值。
                }
            } finally {
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```

## 扩容transfer
transfer扩容操作：
1)单线程构建两倍容量的nextTable；
2)允许多线程复制原table元素到nextTable。

节点从table移动到nextTable，大体思想是遍历、复制的过程:
1. 首先根据运算得到需要遍历的次数i，然后利用tabAt方法获得i位置的元素f，初始化一个forwardNode实例fwd。
2. 如果f == null，则在table中的i位置放入fwd，这个过程是采用Unsafe.compareAndSwapObjectf方法实现的，很巧妙的实现了节点的并发移动。
3. 如果f是链表的头节点，节点上锁（synchronized），就构造一个反序链表，把他们分别放在nextTable的i和i+n的位置上，移动完成，采用Unsafe.putObjectVolatile方法给table原位置赋值fwd。
4. 如果f是TreeBin节点，节点上锁（synchronized），也做一个反序处理，并判断是否需要untreeify，把处理的结果分别放在nextTable的i和i+n的位置上，移动完成，同样采用Unsafe.putObjectVolatile方法给table原位置赋值fwd。

遍历过所有的节点以后就完成了复制工作，把table指向nextTable，并更新sizeCtl为新数组大小的0.75倍 ，扩容完成。

```java
/**
 * Moves and/or copies the nodes in each bin to new table.
 * 将table中每一个bin（桶位）的Node移动或复制到nextTable
 */
private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
    int n = tab.length, stride;
    if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
        stride = MIN_TRANSFER_STRIDE; // subdivide range
    if (nextTab == null) {            // initiating
        try {
            @SuppressWarnings("unchecked")
            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1]; //构造一个nextTable对象 它的容量是原来的两倍
            nextTab = nt;
        } catch (Throwable ex) {      // try to cope with OOME
            sizeCtl = Integer.MAX_VALUE;
            return;
        }
        nextTable = nextTab;
        transferIndex = n;
    }
    int nextn = nextTab.length;
    ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab); // 构造一个连节点指针，用于标志位。
    boolean advance = true; //并发扩容的关键属性。如果等于true，说明这个节点已经处理过。
    boolean finishing = false; // to ensure sweep before committing nextTab
    for (int i = 0, bound = 0;;) {
        Node<K,V> f; int fh;
        //这个while循环体的作用就是在控制i-- ，通过i--可以依次遍历原hash表中的节点
        while (advance) {
            int nextIndex, nextBound;
            if (--i >= bound || finishing)
                advance = false;
            else if ((nextIndex = transferIndex) <= 0) {
                i = -1;
                advance = false;
            }
            else if (U.compareAndSwapInt
                     (this, TRANSFERINDEX, nextIndex,
                      nextBound = (nextIndex > stride ?
                                   nextIndex - stride : 0))) {
                bound = nextBound;
                i = nextIndex - 1;
                advance = false;
            }
        }
        if (i < 0 || i >= n || i + n >= nextn) {
            int sc;
            if (finishing) { // 如果所有的节点都已经完成复制工作，就把nextTable赋值给table，清空临时对象nextTable
                nextTable = null;
                table = nextTab;
                sizeCtl = (n << 1) - (n >>> 1);
                return;
            }

            // 利用CAS方法更新这个扩容阈值，在这里面sizectl值减一，说明新加入一个线程参与到扩容操作
            if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                    return;
                finishing = advance = true;
                i = n; // recheck before commit
            }
        }
        // 如果遍历到的节点为空 则放入ForwardingNode连接点指针
        else if ((f = tabAt(tab, i)) == null)
            advance = casTabAt(tab, i, null, fwd);
		// 如果遍历到ForwardingNode节点，说明这个点已经被处理过了，直接跳过。这里是控制并发扩容的核心
        else if ((fh = f.hash) == MOVED)
            advance = true; // already processed
        else {
        	// 节点上锁
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    Node<K,V> ln, hn;
                    // 如果fh>=0 证明这是一个Node节点
                    if (fh >= 0) {
                        int runBit = fh & n;
                        // 以下的部分在完成的工作是构造两个链表：一个是原链表，另一个是原链表的反序排列
                        Node<K,V> lastRun = f;
                        for (Node<K,V> p = f.next; p != null; p = p.next) {
                            int b = p.hash & n;
                            if (b != runBit) {
                                runBit = b;
                                lastRun = p;
                            }
                        }
                        if (runBit == 0) {
                            ln = lastRun;
                            hn = null;
                        }
                        else {
                            hn = lastRun;
                            ln = null;
                        }
                        for (Node<K,V> p = f; p != lastRun; p = p.next) {
                            int ph = p.hash; K pk = p.key; V pv = p.val;
                            if ((ph & n) == 0)
                                ln = new Node<K,V>(ph, pk, pv, ln);
                            else
                                hn = new Node<K,V>(ph, pk, pv, hn);
                        }
                        // 在nextTable的i位置上插入一个链表
                        setTabAt(nextTab, i, ln);
                        // 在nextTable的i+n的位置上插入另一个链表
                        setTabAt(nextTab, i + n, hn);
                        // 在table的i位置上插入forwardNode节点，表示已经处理过该节点
                        setTabAt(tab, i, fwd);
                        // 设置advance为true，返回到上面的while循环中，就可以执行i--操作
                        advance = true;
                    }
                    // 对TreeBin对象进行处理，与上面的过程类似
                    else if (f instanceof TreeBin) {
                        TreeBin<K,V> t = (TreeBin<K,V>)f;
                        TreeNode<K,V> lo = null, loTail = null;
                        TreeNode<K,V> hi = null, hiTail = null;
                        int lc = 0, hc = 0;
                        // 构造正序和反序两个链表
                        for (Node<K,V> e = t.first; e != null; e = e.next) {
                            int h = e.hash;
                            TreeNode<K,V> p = new TreeNode<K,V>
                                (h, e.key, e.val, null, null);
                            if ((h & n) == 0) {
                                if ((p.prev = loTail) == null)
                                    lo = p;
                                else
                                    loTail.next = p;
                                loTail = p;
                                ++lc;
                            }
                            else {
                                if ((p.prev = hiTail) == null)
                                    hi = p;
                                else
                                    hiTail.next = p;
                                hiTail = p;
                                ++hc;
                            }
                        }
                        // 如果扩容后已经不再需要tree的结构，反向转换为链表结构
                        ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                            (hc != 0) ? new TreeBin<K,V>(lo) : t;
                        hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                            (lc != 0) ? new TreeBin<K,V>(hi) : t;
                        // 在nextTable的i位置上插入一个链表
                        setTabAt(nextTab, i, ln);
                        // 在nextTable的i+n的位置上插入另一个链表
                        setTabAt(nextTab, i + n, hn);
                        // 在table的i位置上插入forwardNode节点，表示已经处理过该节点
                        setTabAt(tab, i, fwd);
                        // 设置advance为true，返回到上面的while循环中，就可以执行i--操作
                        advance = true;
                    }
                }
            }
        }
    }
}
```


* * *
感谢：
http://www.importnew.com/22007.html
http://www.jianshu.com/p/c0642afe03e0
http://www.cnblogs.com/huaizuo/p/5413069.html