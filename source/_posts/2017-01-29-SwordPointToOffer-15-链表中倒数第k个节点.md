---
title: 'SwordPointToOffer(15): 链表中倒数第k个节点'
date: 2017-01-29 13:12:47
categories:
- SwordPointToOffer
---

# 题目
输入一个链表，输出该链表中倒数第k个节点。

# 分析
思路1：先遍历一次链表，得到节点数n，然后遍历输出第(n-k+1)个节点。

思路2：只遍历一次链表，维护两个指针，一个从头开始计数，另一个指针也从头开始计数，只不过两个指针的距离保持在k-1，当第一个指针到达尾节点时，第二个指针就是倒数第k个节点。

需要注意代码的鲁棒性：
1. 校验输入为空
2. 校验链表节点数少于k
3. 校验输入参数k是否大于0

# 实现
```java
public class FindKthToTail {
    class ListNode{
        int value;
        ListNode next;
    }
    
    public ListNode find(ListNode head, int k){
        if (head == null || k <= 0){
            return null;
        }
        
        ListNode index1 = head;
        ListNode index2 = head;
        for (int i = 0; i < k - 1; i++){
            if (index1.next != null){
                index1 = index1.next;
            }else {
                // 节点数少于k个
                return null;
            }
        }
        
        while (index1.next != null){
            index1 = index1.next;
            index2 = index2.next;
        }
        
        return index2;
    }
}
```