---
title: 'SwordPointToOffer(22): 栈的压入、弹出序列'
date: 2017-02-05 09:31:53
categories:
- SwordPointToOffer
---

# 题目
输入两个整数序列，第一个序列表示栈的压入顺序，请判断第二个序列是否为该栈的弹出顺序。假设压入栈的所有数字均不相等。例如，序列1、2、3、4、5是某栈的压入序列，序列4、5、3、2、1是该栈的对应的一个弹出序列，但是4、3、5、1、2就不可能是该栈的弹出序列。

# 分析
思路：借助辅助栈，把压入序列的数字依次压入该辅助栈，并按照弹出序列从该栈中弹出数字。

规律：如果下一个弹出的数字刚好是栈顶数字，那么直接弹出。如果下一个弹出的数字不在栈顶，我们把压栈序列中还没有入栈的数字压入辅助栈，直到把下一个需要弹出的数字压入栈顶为止。如果所有的数字都压入栈了仍然没有找到下一个弹出的数字，那么该序列不可能是一个弹出序列。

# 实现
```java
import java.util.Stack;

public class StackPushPopOrder {
    
    public static void main(String[] args) {
        int[] push = new int[]{1, 2, 3, 4, 5};
        int[] pop1 = new int[]{4, 5, 3, 2, 1};
        int[] pop2 = new int[]{4, 5, 3, 1, 2};

        System.out.println(isPopOrder(push, pop1));
        System.out.println(isPopOrder(push, pop2));
    }
    
    public static boolean isPopOrder(int[] push, int[] pop){
        if (push == null || pop == null || pop.length == 0 || push.length == 0 ||  push.length != pop.length)
            throw new RuntimeException("Invalid input!");
        
        boolean isPossible = false;
        // 辅助栈
        Stack<Integer> stack = new Stack<>();
        int indexForPush = 0;
        int indexForPop = 0;
        while (indexForPop < pop.length){
            while (indexForPush < push.length && (stack.isEmpty() || stack.peek() != pop[indexForPop])){
                stack.push(push[indexForPush]);
                ++indexForPush;                    
            }
            
            if (stack.peek() != pop[indexForPop]){
                break;
            }
            
            stack.pop();
            ++indexForPop;
        }

        if (stack.isEmpty() && indexForPop == pop.length){
            isPossible = true;
        }
        
        return isPossible;
    }
}
```