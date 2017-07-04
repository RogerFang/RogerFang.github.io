---
title: 'SwordPointToOffer(41): 和为s的两个数字VS和为s的连续整数序列(补充)'
date: 2017-02-07 15:51:34
categories:
- SwordPointToOffer
---

# 题目
输入一个正数s，打印出所有和为s的连续正数序列（至少含有两个数）。

例如输入15，由于 1+2+3+4+5 = 4+5+6 = 7+8 = 15，所以输出3个连续序列。

# 分析
考虑用两个数 small 和 big 分别表示序列的最小值和最大值。首先把 small 初始化为 1，big 初始化为 2。如果从 small 到 big 的序列的和大于 s，我们可以从序列中去掉较小的值，也就是增大 small 的值。如果从 small 到 big 的序列的和小于 s，我们可以增大 big，让这个序列包含更多的数字。因为这个序列至少要有两个数字，我们一直增加 small 到(1+s)/2 为止。

# 实现
```java
public class ContinuousSequenceWithSum {
    
    public List<List<Integer>> handle(int sum){
        List<List<Integer>> result = new ArrayList<>();
        if (sum < 3)
            return result;
        
        int small = 1;
        int big = 2;
        int mid = (1 + sum) >> 1;
        int curSum = small + big;
        while (small < mid){
            if (curSum == sum){
                List<Integer> list = new ArrayList<>();
                for (int i = small; i <= big; ++i){
                    list.add(i);
                }
                result.add(list);
            }
            
            while (curSum > sum && small < mid){
                curSum -= small;
                ++small;
                if (curSum == sum){
                    List<Integer> list = new ArrayList<>();
                    for (int i = small; i <= big; ++i){
                        list.add(i);
                    }
                    result.add(list);
                }
            }
            ++big;
            curSum += big;
        }
        return result;
    }
}
```