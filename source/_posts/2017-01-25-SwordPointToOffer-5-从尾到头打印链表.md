---
title: 'SwordPointToOffer(5): 从尾到头打印链表'
date: 2017-01-25 14:55:43
categories:
- SwordPointToOffer
---

# 题目
输入一个链表，从尾到头打印链表每个节点的值。

# 分析
在不改变链表结构的情况下，可以通过**栈**来实现“后进先出”的顺序，也可以使用**递归**来实现。
递归在本质上就是一个栈结构，但是递归可能会导致函数调用层级很深造成栈溢出，显示地用栈基于循环实现的代码更好一些。

# 实现
```java
public class ListReversePrintDemo {

    public ArrayList<Integer> printListFromTailToHead(ListNode listNode) {
        ArrayList<Integer> list = new ArrayList<>();
        // usingRecursion(list, listNode);
        usingIteration(list, listNode);
        return list;
    }

    /**
     * 使用栈实现
     */
    private void usingIteration(ArrayList<Integer> list, ListNode listNode) {
        if (listNode != null) {
            Stack<Integer> stack = new Stack<>();
            while (listNode != null) {
                stack.push(listNode.val);
                listNode = listNode.next;
            }

            while (!stack.isEmpty()) {
                list.add(stack.pop());
            }
        }
    }

    /**
     * 通过递归实现
     */
    private void usingRecursion(ArrayList<Integer> list, ListNode listNode) {
        if (listNode != null) {
            usingRecursion(list, listNode.next);
            list.add(listNode.val);
        }
    }

    public class ListNode {
        int val;
        ListNode next = null;

        ListNode(int val) {
            this.val = val;
        }
    }
}
```