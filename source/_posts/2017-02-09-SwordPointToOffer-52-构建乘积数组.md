---
title: 'SwordPointToOffer(52): 构建乘积数组'
date: 2017-02-09 13:40:23
categories:
- SwordPointToOffer
---

# 题目
给定一个数组 A[0,1,…,n-1]，请构建一个数组 B[0,1,…,n-1]，其中 B 中的元素 B[i]=A[0]×A[1]×…×A[i-1]×A[i+1]×…×A[n-1]，不能使用除法。

# 分析
定义 B[i] = C[i] * D[i]，C[i] = A[0]*A[1]*...*A[i-1]，D[i] = A[i+1]*...*A[n-1]。

C[i]采用从前到后的顺序计算而来，C[i] = C[i-1]*A[i-1]；D[i]采用从后往前的顺序计算出来，D[i] = A[i+1]*D[i+1]。

时间复杂度是$O(n)$。

# 实现
```java
public class ArrayConstruction {
    
    public int[] handle(int[] data){
        if (data== null || data.length <= 0)
            return null;
        int[] result = new int[data.length];
        // 保存C[]累乘值,累乘初始值取1
        result[0] = 1;
        for (int i = 1; i < data.length; ++i){
            result[i] = result[i-1]*data[i-1];
        }
        // 保存D[] 的累乘值
        int tmp = 1;
        for (int i = data.length - 2; i >= 0; --i){
            tmp *= data[i+1];
            result[i] *= tmp;
        }
        return result;
    }
}
```