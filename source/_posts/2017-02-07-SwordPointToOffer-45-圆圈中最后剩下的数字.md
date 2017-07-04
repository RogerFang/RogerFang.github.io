---
title: 'SwordPointToOffer(45): 圆圈中最后剩下的数字'
date: 2017-02-07 20:52:31
categories:
- SwordPointToOffer
tags:
- Josephus Problem
---

# 题目
0,1,...,n-1折n个数字排成一个圆圈，从数字0开始每次从这个圆圈里删除第m个数字。求出这个圆圈里剩下的最后一个数字。

# 分析
## 方法一
> 环形链表模拟圆圈的经典解法。

创建一个总共有 n 个结点的环形链表，然后每次在这个链表中删除第 m 个结点。
时间复杂度$O(nm)$，空间复杂度$O(n)$。

## 方法二
> 分析每次被删除的数字的规律并直接计算出圆圈中最后剩下的数字。

$$ f(n,m)= \begin{cases} 0, & \text {n=1} \\\ (f(n-1,m)+m)\%n, & \text{n>1} \end{cases} $$

# 实现
```java
import java.util.LinkedList;
import java.util.List;

public class LastRemaining {

    public int handle1(int n, int m){
        if (n < 1 || m < 1)
            return -1;

        List<Integer> list = new LinkedList<>();
        for (int i = 0; i < n; ++i)
            list.add(i);
        // 要删除的元素id
        int idx = 0;
        while (list.size() > 1){
            idx = (idx + m - 1)%list.size();
            list.remove(idx);
        }
        return list.get(0);
    }

    public int handle2(int n, int m){
        if (n < 1 || m < 1) {
            return -1;
        }
        int last = 0;
        for (int i = 2; i <=n ; i++) {
            last = (last + m)%i;
        }
        return last;
    }
}
```
