---
title: 'SwordPointToOffer(7) 补充: 用两个队列实现栈'
date: 2017-01-25 21:22:54
categories:
- SwordPointToOffer
---

# 问题
用两个队列来实现一个栈，完成栈的push和pop操作。 栈中的元素为int类型。

# 分析
通过具体的例子抽象整个过程，所有的入栈操作插入到非空的队列上，所有的出栈操作将非空队列的 length-1 个元素转移到另一个空队列中，剩下的最后一个元素出栈。

# 实现
```java
public class QueueForStack {
    Queue<Integer> queue1 = new LinkedList<>();
    Queue<Integer> queue2 = new LinkedList<>();

    public void push(int node){
        if (!queue1.isEmpty()){
            queue1.offer(node);
        }else {
            queue2.offer(node);
        }
    }

    public Integer pop(){
        if (!queue1.isEmpty()){
            while (queue1.size() > 1){
                queue2.offer(queue1.poll());
            }
            return queue1.poll();
        }

        if (!queue2.isEmpty()){
            while (queue2.size() > 1){
                queue1.offer(queue2.poll());
            }
            return queue2.poll();
        }

        return null;
    }
}
```