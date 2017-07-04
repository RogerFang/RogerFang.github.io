---
title: 'SwordPointToOffer(17): 合并两个排序的链表'
date: 2017-01-29 14:59:20
categories:
- SwordPointToOffer
---

# 题目
输入两个递增排序的链表，合并这两个链表并使新链表中的节点仍然是按照递增排序的。

# 分析
合并链表的过程：
1. 比较两个链表头节点的值大小，将较小的节点合并到新链表中；
2. 递归完成所有比较

# 实现
```java
public class MergeList {
    
    class ListNode{
        int value;
        ListNode next;
    }

    public ListNode merge(ListNode list1,ListNode list2) {
        if(list1 == null){
            return list2;
        }else if(list2 == null){
            return list1;
        }

        ListNode mergeList = null;
        if(list1.value < list2.value){
            mergeList = list1;
            mergeList.next = merge(list1.next, list2);
        }else{
            mergeList = list2;
            mergeList.next = merge(list1, list2.next);
        }

        return mergeList;
    }
}
```