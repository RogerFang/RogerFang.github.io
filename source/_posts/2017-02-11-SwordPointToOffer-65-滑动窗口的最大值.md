---
title: 'SwordPointToOffer(65): 滑动窗口的最大值'
date: 2017-02-11 15:40:33
categories:
- SwordPointToOffer
---

# 题目
给定一个数组和滑动窗口的大小，请找出所有滑动窗口里的最大值。

例如：输入数组 {2,3,4,2,6,2,5,1}及滑动窗口的大小3，那么一共存在6个滑动窗口，它们的最大值分别为 {4,4,6,6,6,5}。

# 分析
滑动窗口可以看成是一个队列，当窗口滑动时，处于窗口的第一个数字被删除，同时在窗口的末尾添加一个新的数字。
## 方法一
前面的题21和题7相结合，实现一个包含$O(1)$时间的min或max的栈，同时用两个栈实现一个队列。总的时间复杂度为$O(n)$，但是增加了空间复杂度$O(n)$。

## 方法二
换一种思路，并不把滑动窗口的每个数值都存入队列中，而是只把有可能成为滑动窗口最大值的数值存入到一个两端开口的队列。

# 实现
```java
import java.util.ArrayList;
import java.util.LinkedList;

public class MaxInSlidingWindow {

    public ArrayList<Integer> handle(int[] num, int size) {
        ArrayList<Integer> maxInWindows = new ArrayList<>();
        if (num == null || size < 1 || num.length < size)
            return maxInWindows;
        LinkedList<Integer> queue = new LinkedList<>();
        for (int i = 0; i < size; ++i) {
            while (!queue.isEmpty() && num[i] >= num[queue.getLast()]) {
                queue.removeLast();
            }
            queue.addLast(i);
        }
        for (int i = size; i < num.length; ++i) {
            maxInWindows.add(num[queue.getFirst()]);
            while (!queue.isEmpty() && num[i] >= num[queue.getLast()]) {
                queue.removeLast();
            }
            // 队首元素超出滑动窗口进行出队
            if (!queue.isEmpty() && i - queue.getFirst() >= size)
                queue.removeFirst();
            queue.addLast(i);
        }
        maxInWindows.add(num[queue.getFirst()]);
        return maxInWindows;
    }
}
```