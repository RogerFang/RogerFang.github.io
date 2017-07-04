---
title: 'SwordPointToOffer(56): 链表中环的入口节点'
date: 2017-02-09 22:28:34
categories:
- SwordPointToOffer
---

# 题目
一个链表中包含环，请找出该链表的环的入口结点。
```java
/**
 * 节点定义
 */
public class ListNode {
    int val;
    ListNode next = null;

    ListNode(int val) {
        this.val = val;
    }
}
````

# 分析
可以用两个指针来解决这个问题。先定义两个指针 P1 和 P2 指向链表的头结点。如果链表中环有 n 个结点，指针 P1 在链表上向前移动 n 步，然后两个指针以相同的速度向前移动。当第二个指针指向环的入口结点时，第一个指针已经围绕着环走了一圈又回到了入口结点。

剩下的问题就是**如何得到环中结点的数目**。我们在面试题 15 的第二个相关题目时用到了一快一慢的两个指针。如果两个指针相遇，表明链表中存在环。两个指针相遇的结点一定是在环中。可以从这个结点出发，一边继续向前移动一边计数，当再次回到这个结点时就可以得到环中结点数了。

# 实现
```java
public class EntryNodeInListLoop {

    class ListNode {
        int val;
        ListNode next = null;

        ListNode(int val) {
            this.val = val;
        }
    }
    
    public ListNode handle(ListNode head){
        ListNode meetingNode = getMeetingNode(head);
        if (meetingNode == null)
            return null;
        
        int nodesInLoop = 1;
        ListNode node1 = meetingNode;
        while (node1.next != meetingNode){
            node1 = node1.next;
            ++nodesInLoop;
        }
        
        // move node1
        node1 = head;
        for (int i = 0; i < nodesInLoop; ++i)
            node1 = node1.next;
        
        // move node1 & node2
        ListNode node2 = head;
        while (node1 != node2){
            node1 = node1.next;
            node2 = node2.next;
        }
        return node1;
    }
    
    private ListNode getMeetingNode(ListNode head){
        if (head == null)
            return null;
        ListNode slow = head;
        ListNode fast = slow.next;
        
        while (slow != null && fast != null){
            if (fast == slow)
                return fast;
            slow = slow.next;
            fast = fast.next;
            // fast 走快一点才能和slow相遇
            if (fast != null)
                fast = fast.next;
        }
        return null;
    }
}
```