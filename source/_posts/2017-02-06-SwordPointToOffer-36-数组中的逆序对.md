---
title: 'SwordPointToOffer(36): 数组中的逆序对'
date: 2017-02-06 20:30:24
categories:
- SwordPointToOffer
---

# 题目
在数组中的两个数字如果前面一个数字大于后面的数字，则这两个数字组成一个逆序对。输入一个数组，求出这个数组中的逆序对的总数。

# 分析
## 方法一
从头到尾扫描数组，没扫描到一个数字，就逐个比较该数字和后面的数字的大小，时间复杂度是 $O(n^2)$。

## 方法二
先把数组分隔成子数组， 先统计出子数组内部的逆序对的数目，然后再统计出两个相邻子数组之间的逆序对的数目。在统计逆序对的过程中，还需要对数组进行排序。这个排序的过程实际上就是**归并排序**。

> 该方法会改变原数组，在data和copy之间交替作为辅助数组。

# 实现
```java
public class InversePairs {

    public static void main(String[] args) {
        InversePairs test = new InversePairs();

        int[] data = new int[]{5,4,3,2,1};
        System.out.println(test.handle(data));
    }
    
    public int handle(int[] data){
        if (data == null || data.length < 2)
            throw new RuntimeException("invalid input!");
        
        int[] copy = new int[data.length];
        System.arraycopy(data, 0, copy, 0, data.length);
        return doHandle(data, copy, 0, data.length - 1);
    }
    
    private int doHandle(int[] data, int[] copy, int start, int end){
        if (start == end){
            copy[start] = data[start];
            return 0;
        }
        
        int mid = (end + start) >> 1;
        int left = doHandle(copy, data, start, mid);
        int right = doHandle(copy, data, mid + 1, end);
        
        // 初始化前半段最后一个数字的下标
        int i = mid;
        // 初始化后半段最后一个数字的下标
        int j = end;
        int indexOfCopy = end;
        int count = 0;
        while (i >= start && j >= mid + 1){
            if (data[i] > data[j]){
                copy[indexOfCopy--] = data[i--];
                
                count += j - mid;
            }else {
                copy[indexOfCopy--] = data[j--];
            }
        }
        
        for (; i >= start; --i){
            copy[indexOfCopy--] = data[i];
        }
        for(; j >= mid + 1; --j){
            copy[indexOfCopy--] = data[j];
        }
        return left + right + count;
    }
}
```