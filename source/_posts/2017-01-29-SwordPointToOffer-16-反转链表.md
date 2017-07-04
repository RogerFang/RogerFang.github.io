---
title: 'SwordPointToOffer(16): 反转链表'
date: 2017-01-29 14:41:10
categories:
- SwordPointToOffer
---

# 题目
定义一个函数，输入一个链表的头节点，反转该链表并返回反转后链表的头节点。

# 分析
分析整个调整链表方向的过程，需要借助具体的例子来分析，例如原链表中的某一段h->i->j，当调整h<-i时，将i的next节点指向节点h，但是会导致i->j的断裂，所以需要保存这个引用。

# 实现
```java
public class ReverseList {
    
    class ListNode{
        int value;
        ListNode next;
    }
    
    public ListNode reverse(ListNode head){
        ListNode reverseHead = null;
        ListNode node = head;
        ListNode preNode = null;
        
        while (node != null){
            ListNode nextNode = node.next;
            if (nextNode == null){
                reverseHead = node;
            }
            node.next = preNode;
            preNode = node;
            node = nextNode;
        }
        return reverseHead;
    }
}
```