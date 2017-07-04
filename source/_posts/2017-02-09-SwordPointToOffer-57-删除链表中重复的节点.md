---
title: 'SwordPointToOffer(57): 删除链表中重复的节点'
date: 2017-02-09 22:53:04
categories:
- SwordPointToOffer
---

# 题目
在一个排序的链表中，存在重复的结点，请删除该链表中重复的结点，重复的结点不保留，返回链表头指针。

例如，链表1->2->3->3->4->4->5 处理后为 1->2->5

# 分析
从头遍历整个链表，如果当前结点的值与下一个结点的值相同，那么它们就是重复的结点，都可以被删除。为了保证删除之后的链表仍然是相连的而没有中间断开，我们要把当前的前一个结点和后面值比当前结点的值要大的结点相连。

# 实现
```java
public class DeleteDuplicatedListNode {

    class ListNode {
        int val;
        ListNode next = null;

        ListNode(int val) {
            this.val = val;
        }
    }
    
    public ListNode deleteDuplication(ListNode head){
        if (head == null)
            return null;
        
        // 临时的头节点
        ListNode root = new ListNode(0);
        root.next = head;

        ListNode pre = root;
        ListNode node = head;
        while (node != null && node.next != null){
            if (node.val == node.next.val){
                while (node.next != null && node.next.val == node.val){
                    node = node.next;
                }
                pre.next = node.next;
            }else {
                pre = node;
            }
            node = node.next;
        }
        return root.next;
    }
}
```