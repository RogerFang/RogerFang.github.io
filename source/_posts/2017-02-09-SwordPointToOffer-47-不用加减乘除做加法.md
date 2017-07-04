---
title: 'SwordPointToOffer(47): 不用加减乘除做加法'
date: 2017-02-09 10:23:00
categories:
- SwordPointToOffer
---

# 题目
写一个函数，求两个整数之和，要求在函数体内不得使用 加减乘除 四则运算符号。

# 分析
不能用四则运算，还有位运算。

比如： `5 + 17 = 22`
5 的二进制是`101`，17 的二进制是`10001`。把计算分成三步：
1. 二进制各位相加但不进位，得到结果是`10100`（最后一位两个都是1，相加结果不计进位）。
2. 记下进位，上一步产生的进位是由最后两个1相加得到的`10`。
3. 将前两步相加（仍重复前两步），得到结果`10110`，也就是22。

关键：
1. 异或运算，相加不进位
2. 与运算，然后再左移一位，只有1和1相加进位

# 实现
```java
public class AddTwoNumber {

    public static void main(String[] args) {
        System.out.println(add(13, 20));
    }
    
    public static int add(int num1, int num2){
        int sum = 0;
        int carry = 0;

        do {
            sum = num1 ^ num2;
            carry = (num1 & num2) << 1;
            num1 = sum;
            num2 = carry;
        }while (carry != 0);
        return sum;
    }
}
```