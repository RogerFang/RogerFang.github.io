---
title: 'leetcode(136/137/260): single number'
date: 2017-07-01 16:19:10
categories:
- leetcode
tags:
- leetcode
---

# 136
Given an array of integers, every element appears **twice** except for one. Find that single one.

Note:
Your algorithm should have a linear runtime complexity. Could you implement it without using extra memory?

```java
public class Solution {
    public int singleNumber(int[] A) {
        int result = 0;
        for (int i : A){
            result ^= i;
        }
        return result;
    }
}
```

# 137
Given an array of integers, every element appears **three times** except for one, which appears exactly once. Find that single one.

Note:
Your algorithm should have a linear runtime complexity. Could you implement it without using extra memory?

```java
public class Solution {
    public int singleNumber(int[] A) {
        int ones = 0, twos = 0;
        for (int a : A){
            ones = (ones ^ a) & ~twos;
            twos = (twos ^ a) & ~ones;
        }
        return ones;
    }
}
```

# 260
Given an array of numbers nums, in which *exactly two elements appear only once and all the other elements appear exactly twice*. Find the two elements that appear only once.

For example:

Given nums = [1, 2, 1, 3, 2, 5], return [3, 5].

Note:
The order of the result is not important. So in the above example, [5, 3] is also correct.
Your algorithm should run in linear runtime complexity. Could you implement it using only constant space complexity?

```java
public class Solution {
    public int[] singleNumber(int[] nums) {
        int xor = 0;
        for (int i : nums){
            xor ^= i;
        }

        int bit = xor & ~(xor - 1);
        int num1 = 0;
        int num2 = 0;
        for (int i : nums){
            if ((i & bit) > 0){
                num1 ^= i;
            } else{
                num2 ^= i;
            }
        }
        return new int[]{num1, num2};
    }
}
```
参见：[剑指offer: 数组中只出现一次的数字](https://rogerfang.github.io/2017/02/07/SwordPointToOffer-40-%E6%95%B0%E7%BB%84%E4%B8%AD%E5%8F%AA%E5%87%BA%E7%8E%B0%E4%B8%80%E6%AC%A1%E7%9A%84%E6%95%B0%E5%AD%97/)