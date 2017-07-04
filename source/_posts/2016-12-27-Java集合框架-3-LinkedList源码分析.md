---
title: 'Java集合框架(3): LinkedList源码分析'
date: 2016-12-27 21:11:55
categories:
- Java集合框架
tags:
- LinkedList
---

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
```

# 简介
1. LinkedList是基于双向循环链表（从源码中可以很容易看出）实现的，实现了List和Deque接口，除了可以当做链表来操作外，它还可以当做栈、队列和双端队列来使用。
2. LinkedList同样是非线程安全的，只在单线程下适合使用。
3. LinkedList实现了Serializable接口，因此它支持序列化，能够通过序列化传输，实现了Cloneable接口，能被克隆。

# 源码解析
## 属性
```java
	// LinkedList中元素的个数
    transient int size = 0;
    // 链表的首节点
    // (first == null && last == null) || (first.prev == null && first.item != null)
    transient Node<E> first;
    // 链表的尾节点
    // (first == null && last == null) || (last.next == null && last.item != null)
    transient Node<E> last;
```

## 构造方法
```java
public LinkedList() {
}

/**
 * 通过一个集合初始化LinkedList
 * 元素顺序由这个结合的迭代器返回顺序决定
 */
public LinkedList(Collection<? extends E> c) {
	this();
	addAll(c);
}
```

## 方法
### linkFirst
```java
/**
 * 使用参数e作为首节点
 */
private void linkFirst(E e) {
	// 得到原首节点
    final Node<E> f = first;
    // 创建新节点，newNode的prev节点为null，元素为e，next节点为原首节点f
    final Node<E> newNode = new Node<>(null, e, f);
    // 首节点设为新节点
    first = newNode;
    if (f == null)
    	// 如果原首节点为空，则尾节点就是首节点，也设为新节点
        last = newNode;
    else
    	// 如果原首节点不为空，则原首节点的prev为新节点
        f.prev = newNode;
    size++;
    modCount++;
}
```

### linkLast
```java
/**
 * 使用参数e作尾节点
 */
void linkLast(E e) {
	// 得到原尾节点
    final Node<E> l = last;
    // 创建新节点，newNode的prev节点为原尾节点，元素为e，next节点为null
    final Node<E> newNode = new Node<>(l, e, null);
    // 尾节点设置为新节点
    last = newNode;
    if (l == null)
    	// 如果原尾节点为空，则首节点就是尾节点，也设为新节点
        first = newNode;
    else
    	// 如果原尾节点不为空，则原尾节点的next为新节点
        l.next = newNode;
    size++;
    modCount++;
}
```

### linkBefore
```java
/**
 * 在指定节点succ前插入新元素，指定节点succ不能为空
 */
void linkBefore(E e, Node<E> succ) {
    // assert succ != null;
    // 获取指定节点的prev节点
    final Node<E> pred = succ.prev;
    // 创建新节点，prev为指定节点的前一个节点，元素为e，next节点为指定节点
    final Node<E> newNode = new Node<>(pred, e, succ);
    // 指定节点succ向前指向新的节点
    succ.prev = newNode;
    if (pred == null)
    	// 如果指定节点的prev节点为空，则新的节点就是首节点
        first = newNode;
    else
    	// 如果指定节点存在prev节点pred，则pred向后指向新节点
        pred.next = newNode;
    size++;
    modCount++;
}
```

### unlinkFirst
```java
/**
 * 删除非空的首节点f并返回首节点元素值
 */
private E unlinkFirst(Node<E> f) {
    // assert f == first && f != null;
    // 获取首节点f的元素值
    final E element = f.item;
    // 得到首节点f的next节点
    final Node<E> next = f.next;
    f.item = null;
    f.next = null; // help GC
    // f的next节点作为首节点
    first = next;
    if (next == null)
        last = null;
    else
        next.prev = null;
    size--;
    modCount++;
    return element;
}
```

### unlinkLast
```java
/**
 * 删除非空的尾节点l并返回尾节点元素值
 */
private E unlinkLast(Node<E> l) {
    // assert l == last && l != null;
    final E element = l.item;
    final Node<E> prev = l.prev;
    l.item = null;
    l.prev = null; // help GC
    last = prev;
    if (prev == null)
        first = null;
    else
        prev.next = null;
    size--;
    modCount++;
    return element;
}
```

### unlink
```java
/**
 * 删除指定非空节点x，并返回其元素值
 */
E unlink(Node<E> x) {
    // assert x != null;
    final E element = x.item;
    final Node<E> next = x.next;
    final Node<E> prev = x.prev;

    if (prev == null) {
        first = next;
    } else {
        prev.next = next;
        x.prev = null;
    }

    if (next == null) {
        last = prev;
    } else {
        next.prev = prev;
        x.next = null;
    }

    x.item = null;
    size--;
    modCount++;
    return element;
}
```

### getFirst
```java
/**
 * 获取第一个元素
 */
public E getFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return f.item;
}
```

### getLast
```java
/**
 * 获取最后一个元素
 */
public E getLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return l.item;
}
```

### removeFirst
```java
/**
 * 删除第一个节点并返回其元素值
 */
public E removeFirst() {
    final Node<E> f = first;
    if (f == null)
        throw new NoSuchElementException();
    return unlinkFirst(f);
}
```

### removeLast
```java
/**
 * 删除最后一个节点并返回其元素值
 */
public E removeLast() {
    final Node<E> l = last;
    if (l == null)
        throw new NoSuchElementException();
    return unlinkLast(l);
}
```

### addFirst
```java
/**
 * 插入一个元素作为第一个元素
 */
public void addFirst(E e) {
    linkFirst(e);
}
```
### addLast
```java
/**
 * 插入一个元素作为最后一个元素
 */
public void addLast(E e) {
    linkLast(e);
}
```

### indexOf
```java
/**
 * 获取指定元素首次出现的索引位置，不存在返回-1
 * 从first开始
 */
public int indexOf(Object o) {
    int index = 0;
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null)
                return index;
            index++;
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item))
                return index;
            index++;
        }
    }
    return -1;
}
```

### lastIndexOf
```java
/**
 * 获取指定元素首次出现的索引位置，不存在返回-1
 * 从last开始
 */
public int lastIndexOf(Object o) {
    int index = size;
    if (o == null) {
        for (Node<E> x = last; x != null; x = x.prev) {
            index--;
            if (x.item == null)
                return index;
        }
    } else {
        for (Node<E> x = last; x != null; x = x.prev) {
            index--;
            if (o.equals(x.item))
                return index;
        }
    }
    return -1;
}
```

### node
```java
/**
 * 获取指定位置的节点
 */
Node<E> node(int index) {
    // assert isElementIndex(index);
	// 如果位置索引小于size的一半(或一半减一)，从first开始遍历
    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
}
```

### contains
```java
public boolean contains(Object o) {
    return indexOf(o) != -1;
}
```

### size
```java
public int size() {
    return size;
}
```

### add
```java
/**
 * 添加一个元素，默认添加到末尾
 */
public boolean add(E e) {
    linkLast(e);
    return true;
}

/**
 * 添加一个元素到指定的索引位置
 * 如果索引值等于size，则添加到末尾；
 * 如果索引值不等于size，插入到该索引原节点之前。
 */
public void add(int index, E element) {
    checkPositionIndex(index);

    if (index == size)
        linkLast(element);
    else
        linkBefore(element, node(index));
}
```

### remove
```java
/**
 * 删除指定元素，默认从first开始，删除第一次出现的元素
 */
public boolean remove(Object o) {
    if (o == null) {
        for (Node<E> x = first; x != null; x = x.next) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        for (Node<E> x = first; x != null; x = x.next) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}

/**
 * 删除指定索引的元素，并返回原索引值
 */
public E remove(int index) {
    checkElementIndex(index);
    return unlink(node(index));
}
```

### addAll
```java
/**
 * 添加指定集合的元素到LinkedList，默认从最后开始添加
 */
public boolean addAll(Collection<? extends E> c) {
    return addAll(size, c);
}

/**
 * 从指定位置开始添加集合
 */
public boolean addAll(int index, Collection<? extends E> c) {
	// 0<= index <= size
    checkPositionIndex(index);

    Object[] a = c.toArray();
    int numNew = a.length;
    if (numNew == 0)
        return false;
	// 插入集合所需的pred节点和succ节点
    Node<E> pred, succ;
    if (index == size) {
    	// 插入位置index原先没有节点
        succ = null;
        pred = last;
    } else {
    	// 插入位置index原先有节点，则pred节点等于当前节点的prev节点，当前节点后移作为succ节点
        succ = node(index);
        pred = succ.prev;
    }

    for (Object o : a) {
        @SuppressWarnings("unchecked") E e = (E) o;
        Node<E> newNode = new Node<>(pred, e, null);
        if (pred == null)
            first = newNode;
        else
            pred.next = newNode;
        // 新加的节点作为下一次添加节点的pred节点
        pred = newNode;
    }

    if (succ == null) {
        last = pred;
    } else {
        pred.next = succ;
        succ.prev = pred;
    }

    size += numNew;
    modCount++;
    return true;
}
```

### clear
```java
public void clear() {
    // Clearing all of the links between nodes is "unnecessary", but:
    // - helps a generational GC if the discarded nodes inhabit
    //   more than one generation
    // - is sure to free memory even if there is a reachable Iterator
    for (Node<E> x = first; x != null; ) {
        Node<E> next = x.next;
        x.item = null;
        x.next = null;
        x.prev = null;
        x = next;
    }
    first = last = null;
    size = 0;
    modCount++;
}
```

### get
```java
/**
 * 获取指定索引的元素值
 */
public E get(int index) {
    checkElementIndex(index);
    return node(index).item;
}
```

### set
```java
/**
 * 修改指定索引的元素值病返回之前的值
 */
public E set(int index, E element) {
    checkElementIndex(index);
    Node<E> x = node(index);
    E oldVal = x.item;
    x.item = element;
    return oldVal;
}
```

### isElementIndex
```java
/**
 * 检查索引是否超出了范围，0<=index<size
 */
private boolean isElementIndex(int index) {
    return index >= 0 && index < size;
}
```

### isPositionIndex
```java
/**
 * 检查位置是否超出范围，0<=index<=size
 */
private boolean isPositionIndex(int index) {
    return index >= 0 && index <= size;
}
```

### outOfBoundsMsg
```java
/**
 * IndexOutOfBoundsException异常详情
 */
private String outOfBoundsMsg(int index) {
    return "Index: "+index+", Size: "+size;
}
```

### checkElementIndex
```java
/**
 * 检查元素索引是否超出范围
 */
private void checkElementIndex(int index) {
    if (!isElementIndex(index))
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```

### checkPositionIndex
```java
/**
 * 检查位置是否超出范围
 */
private void checkPositionIndex(int index) {
    if (!isPositionIndex(index))
        throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```

- - -

> Queue Operations: 提供普通队列和双向队列的功能，当然也可以实现栈，FIFO/FILO。

### peek获取队首元素
```java
/**
 * 出队
 * 1.从前端
 * 2.**不会删除**元素（节点）
 * 3.不存在**返回null**
 */
public E peek() {
    final Node<E> f = first;
    return (f == null) ? null : f.item;
}
```

### element获取队首元素
```java
/**
 * 出队
 * 1.从前端
 * 2.**不会删除**元素（节点）
 * 3.不存在则**抛出异常**
 */
public E element() {
    return getFirst();
}
```

### poll出队
```java
/**
 * 出队
 * 1.从前端
 * 2.会**删除元素**（节点）
 * 3.不存在**返回null**
 */
public E poll() {
    final Node<E> f = first;
    return (f == null) ? null : unlinkFirst(f);
}
```

### remove出队
```java
/**
 * 出队
 * 1.从前端
 * 2.会**删除元素**
 * 3.不存在**抛出异常**
 */
public E remove() {
    return removeFirst();
}
```

### offer入队
```java
/**
 * 入队
 * 从后端
 * 始终返回true
 */
public boolean offer(E e) {
    return add(e);
}
```

### offerFirst入队
```java
/**
 * 入队
 * 从前端
 * 始终返回true
 */
public boolean offerFirst(E e) {
    addFirst(e);
    return true;
}
```

### offerLast
```java
/**
 * 入队
 * 从后端
 * 始终返回true
 */
public boolean offerLast(E e) {
    addLast(e);
    return true;
}
```

### peekFirst获取队首元素
```java
/**
 * 出队
 * 1.从前端
 * 2.不存在返回null
 * 3.不会删除元素
 */
public E peekFirst() {
    final Node<E> f = first;
    return (f == null) ? null : f.item;
 }
```

### peekLast获取栈队尾元素
```java
/**
 * 出队
 * 1.从后端
 * 2.不存在返回null
 * 3.不会删除元素
 */
public E peekLast() {
    final Node<E> l = last;
    return (l == null) ? null : l.item;
}
```

### pollFirst出队
```java
/**
 * 出队
 * 1.从前端
 * 2.不存在返回null
 * 3.会删除元素
 */
public E pollFirst() {
    final Node<E> f = first;
    return (f == null) ? null : unlinkFirst(f);
}
```

### pollLast出队
```java
/**
 * 出队
 * 1.从后端
 * 2.不存在返回null
 * 3.会删除元素
 */
public E pollLast() {
    final Node<E> l = last;
    return (l == null) ? null : unlinkLast(l);
}
```

- - -

> 栈操作: 通过LinkedList实现栈

### push入栈
```java
/**
 * 入栈
 * 从前面添加
 */
public void push(E e) {
    addFirst(e);
}
```

### pop出栈
```java
/**
 * 出栈
 * 返回栈顶元素，从前面删除，不存在抛出异常
 */
public E pop() {
    return removeFirst();
}
```

### removeFirstOccurrence
```java
/**
 *Removes the first occurrence of the specified element in this
 *list (when traversing the list from head to tail).  If the list
 *does not contain the element, it is unchanged.
 */
public boolean removeFirstOccurrence(Object o) {
    return remove(o);
}
```

### removeLastOccurrence
```java
/**
 * Removes the last occurrence of the specified element in this
 * list (when traversing the list from head to tail).  If the list
 * does not contain the element, it is unchanged.
 */
public boolean removeLastOccurrence(Object o) {
    if (o == null) {
        for (Node<E> x = last; x != null; x = x.prev) {
            if (x.item == null) {
                unlink(x);
                return true;
            }
        }
    } else {
        for (Node<E> x = last; x != null; x = x.prev) {
            if (o.equals(x.item)) {
                unlink(x);
                return true;
            }
        }
    }
    return false;
}
```

### 迭代器
```java
public ListIterator<E> listIterator(int index) {
    checkPositionIndex(index);
    return new ListItr(index);
}

private class ListItr implements ListIterator<E> {
    private Node<E> lastReturned;
    private Node<E> next;
    private int nextIndex;
    private int expectedModCount = modCount;

    ListItr(int index) {
        // assert isPositionIndex(index);
        next = (index == size) ? null : node(index);
        nextIndex = index;
    }

    public boolean hasNext() {
        return nextIndex < size;
    }

    public E next() {
        checkForComodification();
        if (!hasNext())
            throw new NoSuchElementException();

        lastReturned = next;
        next = next.next;
        nextIndex++;
        return lastReturned.item;
    }

    public boolean hasPrevious() {
        return nextIndex > 0;
    }

    public E previous() {
        checkForComodification();
        if (!hasPrevious())
            throw new NoSuchElementException();

        lastReturned = next = (next == null) ? last : next.prev;
        nextIndex--;
        return lastReturned.item;
    }

    public int nextIndex() {
        return nextIndex;
    }

    public int previousIndex() {
        return nextIndex - 1;
    }

    public void remove() {
        checkForComodification();
        if (lastReturned == null)
            throw new IllegalStateException();

        Node<E> lastNext = lastReturned.next;
        unlink(lastReturned);
        if (next == lastReturned)
            next = lastNext;
        else
            nextIndex--;
        lastReturned = null;
        expectedModCount++;
    }

    public void set(E e) {
        if (lastReturned == null)
            throw new IllegalStateException();
        checkForComodification();
        lastReturned.item = e;
    }

    public void add(E e) {
        checkForComodification();
        lastReturned = null;
        if (next == null)
            linkLast(e);
        else
            linkBefore(e, next);
        nextIndex++;
        expectedModCount++;
    }

    public void forEachRemaining(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        while (modCount == expectedModCount && nextIndex < size) {
            action.accept(next.item);
            lastReturned = next;
            next = next.next;
            nextIndex++;
        }
        checkForComodification();
    }

    final void checkForComodification() {
        if (modCount != expectedModCount)
            throw new ConcurrentModificationException();
    }
}


public Iterator<E> descendingIterator() {
    return new DescendingIterator();
}

/**
 * Adapter to provide descending iterators via ListItr.previous
 */
private class DescendingIterator implements Iterator<E> {
    private final ListItr itr = new ListItr(size());
    public boolean hasNext() {
        return itr.hasPrevious();
    }
    public E next() {
        return itr.previous();
    }
    public void remove() {
        itr.remove();
    }
}
```

### clone
```java
/**
 * 浅拷贝
 */
public Object clone() {
    LinkedList<E> clone = superClone();

    // Put clone into "virgin" state
    clone.first = clone.last = null;
    clone.size = 0;
    clone.modCount = 0;

    // Initialize clone with our elements
    for (Node<E> x = first; x != null; x = x.next)
        clone.add(x.item);

    return clone;
}

private LinkedList<E> superClone() {
    try {
        return (LinkedList<E>) super.clone();
    } catch (CloneNotSupportedException e) {
        throw new InternalError(e);
    }
}
```

### toArray
```java
public Object[] toArray() {
    Object[] result = new Object[size];
    int i = 0;
    for (Node<E> x = first; x != null; x = x.next)
        result[i++] = x.item;
    return result;
}

public <T> T[] toArray(T[] a) {
    if (a.length < size)
        a = (T[])java.lang.reflect.Array.newInstance(
                            a.getClass().getComponentType(), size);
    int i = 0;
    Object[] result = a;
    for (Node<E> x = first; x != null; x = x.next)
        result[i++] = x.item;

    if (a.length > size)
        a[size] = null;

    return a;
}
```

### 序列化
```java
private void writeObject(java.io.ObjectOutputStream s)
    throws java.io.IOException {
    // Write out any hidden serialization magic
    s.defaultWriteObject();

    // Write out size
    s.writeInt(size);

    // Write out all elements in the proper order.
    for (Node<E> x = first; x != null; x = x.next)
        s.writeObject(x.item);
}

private void readObject(java.io.ObjectInputStream s)
    throws java.io.IOException, ClassNotFoundException {
    // Read in any hidden serialization magic
    s.defaultReadObject();

    // Read in size
    int size = s.readInt();

    // Read in all elements in the proper order.
    for (int i = 0; i < size; i++)
        linkLast((E)s.readObject());
}
```

## 节点
```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;

    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```

* * *
感谢：
http://blog.csdn.net/ns_code/article/details/35787253
http://blog.csdn.net/anxpp/article/details/51203591
