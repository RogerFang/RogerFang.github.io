---
title: 'SwordPointToOffer(10): 二进制中1的个数'
date: 2017-01-26 12:16:47
categories:
- SwordPointToOffer
---

# 题目
输入一个整数，输出该数二进制表示中1的个数。其中负数用补码表示。

# 分析
方法1：用标志位与数target进行按位与操作，如果不为0代表有一个1，循环比较，标志位左移1位。时间复杂度为O(n)，n为二进制的位数。

方法2：将数target减1，然后与原数target进行按位与操作并更新target数，直到target数变为0（每次按位与操作从右到左减少一个二进制位中的1）。时间复杂度为O(m)，m为二进制中1的个数。（target不为0时，至少有一个1）

# 实现
```java
public class BitCount {

    // O(n) n为二进制位数
    public int solutuon1(int n) {
        int flag = 1;
        int count = 0;
        while(flag != 0){
            if( (n&flag) != 0){
                ++count;
            }
            flag = flag << 1;
        }
        return count;
    }

    //O(m) m位1的个数
    public int solution2(int n){
        int count = 0;
        while (n != 0){
            ++count;
            n = (n-1)&n;
        }
        return count;
    }
}
```