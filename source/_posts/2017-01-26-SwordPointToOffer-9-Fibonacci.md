---
title: 'SwordPointToOffer(9): Fibonacci'
date: 2017-01-26 11:11:37
tags:
---

# 拓展
下面代码的时间复杂度是O(n)，但是运用矩阵运算公式及乘方的性质可以达到O(logn)。

青蛙跳台阶问题：一只青蛙一次可以跳1级台阶，也可以跳2级台阶，求跳上n级台阶总共有多少种跳法。
> n>2时，考虑第一次跳1级时，总共有f(n-1)种跳法；第一次跳2级时，总共有f(n-2)种跳法。

# 实现
```java
public class Fibonacci {
    public int fibonacci(int n) {
        if(n == 0){
            return 0;
        }

        if(n == 1){
            return 1;
        }

        int f1 = 0;
        int f2 = 1;
        int fn = 0;
        for(int i = 2; i <= n; ++i){
            fn = f1 + f2;
            f1 = f2;
            f2 = fn;
        }
        return fn;
    }
}
```