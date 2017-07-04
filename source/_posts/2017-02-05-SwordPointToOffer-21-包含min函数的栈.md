---
title: 'SwordPointToOffer(21): 包含min函数的栈'
date: 2017-02-05 08:47:04
categories:
- SwordPointToOffer
---

# 题目
定义栈的数据结构，请在该类型中实现一个能够得到栈的最小元素的min函数。在该栈中，调用min、push以及pop的时间复杂度都是O(1)。

# 分析
如果在定义的数据结构中，仅仅通过添加一个全局变量来保存最小元素，当最小元素出栈时，则得不到下一个最小元素。

思路：通过一个辅助栈来保存每次应该存放的最小元素，当前最小元素是辅助栈的栈顶元素。当数据栈进行出栈操作时，辅助栈也跟着进行出栈操作。如果入栈的元素大于辅助栈的栈顶元素，则重复将原辅助栈的栈顶元素进行入栈。

# 实现
```java
import java.util.Stack;

public class MinInStack {
    public static void main(String[] args) {
        MyStack<Integer> stack = new MyStack<>();
        stack.push(3);
        stack.push(4);
        stack.push(2);
        stack.push(1);

        System.out.println(stack.min());
        stack.pop();
        System.out.println(stack.min());
        stack.pop();
        System.out.println(stack.min());
        stack.pop();
        System.out.println(stack.min());
    }
    
    static class MyStack<T extends Comparable<T>>{
        // 数据栈
        private Stack<T> dataStack;
        // 辅助栈
        private Stack<T> minStack;

        public MyStack() {
            dataStack = new Stack<T>();
            minStack = new Stack<T>();
        }
        
        public void push(T element){
            if (element == null){
                throw new RuntimeException("Element cannot be null!");
            }
            
            if (dataStack.isEmpty()){
                minStack.push(element);
            }else {
                T curMinEle = minStack.peek();
                if (element.compareTo(curMinEle) < 0){
                    minStack.push(element);
                }else {
                    minStack.push(curMinEle);
                }
            }
            
            dataStack.push(element);
        }
        
        public T pop(){
            if (dataStack.isEmpty()){
                throw new RuntimeException("The Stack is empty!");
            }
            
            minStack.pop();
            return dataStack.pop();
        }
        
        public T min(){
            if (dataStack.isEmpty()){
                throw new RuntimeException("The Stack is empty!");
            }
            
            return minStack.peek();
        }
    }
}
```