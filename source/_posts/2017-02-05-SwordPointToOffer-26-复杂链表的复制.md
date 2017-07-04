---
title: 'SwordPointToOffer(26): 复杂链表的复制'
date: 2017-02-05 13:05:07
categories:
- SwordPointToOffer
---

# 题目
输入一个复杂链表（每个节点中有节点值，以及两个指针，一个指向下一个节点，另一个特殊指针指向任意一个节点或者为`null`），返回结果为复制后复杂链表的head。

# 分析
## 方法一
> 时间复杂度是 $O(n_2)$。

思路：
1. 复制原始链表上的每个节点，并用`next`连接起来；
2. 设置节点的`random`指针，在原始链表中定位`random`指向的节点，从头节点开始查找经过`s`步，然后在新链表中也从头节点开始，经过`s`步就是对应的节点。

## 方法二
> 空间复杂度是 $O(n)$，时间复杂度是$O(n)$。

思路：
1. 复制原始链表上的每个节点，并用`next`连接起来；
2. 在复制节点的时候，用**哈希表**把`<N, N'>`的配对信息存储起来；
3. 利用哈希表来设置`random`指针的时间复杂度为$O(n)$。

## 方法三
> 时间复杂度是$O(n)$，不需要辅助空间。

思路：
1. 复制原始链表上的每个节点，并每个复制的节点链接在原节点的后面；
2. 设置节点的`random`指针，假设原节点 N 的`random`指向节点 S，则其对应的复制节点 N' 的`random`指向节点 S的下一个节点 S'；
3. 拆分链表，把奇数位的节点链接起来就是原始链表，把偶数位的节点链接起来就是复制链表。

**Step1**:
![](/images/swordpointtooffer/t26_1.png)

**Step2**:
![](/images/swordpointtooffer/t26_2.png)

**Step3**:
![](/images/swordpointtooffer/t26_3.png)

# 实现
```java
public class CopyComplexList {

    class RandomListNode {
        int label;
        RandomListNode next = null;
        RandomListNode random = null;

        RandomListNode(int label) {
            this.label = label;
        }
    }
    
    public RandomListNode clone(RandomListNode head){
        if (head == null)
            return null;
        
        cloneNode(head);
        connectRandomNodes(head);
        return reconnectNodes(head);
    }
    
    private void cloneNode(RandomListNode head){
        RandomListNode node = head;
        while (node != null){
            RandomListNode clonedNode = new RandomListNode(node.label);
            RandomListNode nextNode = node.next;
            node.next = clonedNode;
            clonedNode.next = nextNode;
            clonedNode.random = null;
            
            node = clonedNode.next;
        }
    }
    
    private void connectRandomNodes(RandomListNode head){
        RandomListNode node = head;
        while (node != null){
            RandomListNode clonedNode = node.next;
            if (node.random != null){
                clonedNode.random = node.random.next;
            }
            node = clonedNode.next;
        }
    }
    
    private RandomListNode reconnectNodes(RandomListNode head){
        RandomListNode node = head;
        RandomListNode clonedHead = null;
        RandomListNode clonedNode = null;
        
        if (head != null){
            clonedHead = clonedNode = node.next;
            node.next = clonedNode.next;
            node = node.next;
        }
        while (node != null){
            clonedNode.next = node.next;
            clonedNode = clonedNode.next;
            node.next = clonedNode.next;
            node = node.next;
        }
        return clonedHead;
    }
}
```