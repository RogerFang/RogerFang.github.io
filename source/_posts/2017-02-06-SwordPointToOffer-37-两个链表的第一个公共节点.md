---
title: 'SwordPointToOffer(37): 两个链表的第一个公共节点'
date: 2017-02-06 22:42:11
categories:
- SwordPointToOffer
---

# 题目
输入两个链表，找出它们的第一个公共节点。

# 分析
## 方法一
蛮力法：在第一个链表上顺序遍历每个节点，每遍历到一个节点的时候，在第二个链表上顺序遍历每个节点。如果在第二个链表上有一个节点和第一个链表上的节点一样，说明两个链表在这个节点上重合。

时间复杂度是$O(mn)$，m和n分别为两个链表的长度。

## 方法二
两个单链表有公共节点时，呈 Y 字型。
借助两个辅助栈，从链表的尾部开始比较，最后一个相同的节点就是要找的节点。
时间复杂度是$O(m + n)$，空间复杂度是$O(m + n)$。

## 方法三
不借助辅助栈，时间复杂度也是$O(m + n)$。

首先遍历两个链表得到各自的长度m和n，然后长一点的链表先遍历(m-n)步，然后两个链表同时继续遍历并比较节点是否相等。

# 实现
```java 方法三
public class FindFirstCommonNode {

    class ListNode {
        int val;
        ListNode next = null;

        ListNode(int val) {
            this.val = val;
        }
    }
    
    public ListNode handle(ListNode head1, ListNode head2){
        if (head1 == null || head2 == null)
            throw new RuntimeException("invalid input!");
        
        int length1 = getListLength(head1);
        int length2 = getListLength(head2);
        
        ListNode nodeOfLong = head1;
        ListNode nodeOfShort = head2;
        int lengthDiff = length1 - length2;
        if (length1 < length2){
            nodeOfLong = head2;
            nodeOfShort = head1;
            lengthDiff = length2 - length1;
        }
        
        for (int i = 0; i < lengthDiff; i++){
            nodeOfLong = nodeOfLong.next;
        }
        
        while (nodeOfLong != null && nodeOfShort != null && nodeOfLong != nodeOfShort){
            nodeOfLong = nodeOfLong.next;
            nodeOfShort = nodeOfShort.next;
        }
        return nodeOfLong;
    }
    
    private int getListLength(ListNode head){
        int count = 0;
        ListNode node = head;
        while (node != null){
            count++;
            node = node.next;
        }
        return count;
    }
}
```