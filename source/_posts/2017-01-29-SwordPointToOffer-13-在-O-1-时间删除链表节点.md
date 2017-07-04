---
title: 'SwordPointToOffer(13): 在 O(1) 时间删除链表节点'
date: 2017-01-29 11:52:47
tags:
---

# 题目
给定单向链表的头指针和一个节点的指针，定义一个函数在O(1)时间删除该节点。

# 分析
常规的做法是：从链表的头节点开始，顺序遍历查找到要删除的节点，并在链表中删除该节点。
顺序查找使得时间复杂度是O(n)。

实现O(1)时间复杂度：使用复制覆盖的思想，将targetNode节点的next节点的值复制给targetNode节点，并将next节点的下一节点覆盖掉targetNode的next节点，然后删除targetNode节点的原next节点即可。

存在的问题：
1. 对于尾节点，其next节点为null，因而仍需要顺序查找删除节点。
2. 对于不存在的节点，需要O(n)的时间判断节点是否在链表中，所以要实现O(1)时间复杂度的删除操作，需要将检验节点是否在链表中的责任交给delete操作的调用者。

# 实现
```java
class ListNode{
    int value;
    ListNode next;
}

/**
 * 通过复制覆盖来删除节点
 * @param head
 * @param targetNode
 * @return
 */
public ListNode delete(ListNode head, ListNode targetNode){
    if (head == null || targetNode == null){
        return head;
    }

    if (head == targetNode){
        return head.next;
    }

    if (targetNode.next == null){
        ListNode tmp = head;
        while (tmp.next != targetNode){
            tmp = tmp.next;
        }
        tmp.next = null;
    }else {
        targetNode.value = targetNode.next.value;
        targetNode.next = targetNode.next.next;
    }

    return head;
}
```