---
title: 'SwordPointToOffer(7): 用两个栈实现队列'
date: 2017-01-25 20:56:45
categories:
- SwordPointToOffer
---

# 题目
用两个栈来实现一个队列，完成队列的enqueue和dequeue操作。 队列中的元素为int类型。

# 分析
通过具体的例子来抽象出整个过程，所有的入队操作插入到stack1中，所有的出队操作从stack2中删除。

# 实现
```java
public class StackForQueue {
    Stack<Integer> stack1 = new Stack<>();
    Stack<Integer> stack2 = new Stack<>();

    public void enqueue(int node) {
        stack1.push(node);
    }

    public int dequeue() {
        if(stack2.size() <= 0){
            while(stack1.size() > 0){
                Integer tmp = stack1.pop();
                stack2.push(tmp);
            }
        }

        if(stack2.size() == 0){
            throw new RuntimeException("The queue is empty");
        }

        return stack2.pop();
    }
}
```
