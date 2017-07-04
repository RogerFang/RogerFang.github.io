---
title: 'SwordPointToOffer(38): 数字在排序数组中出现的次数'
date: 2017-02-07 12:19:27
categories:
- SwordPointToOffer
---

# 题目
统计一个数字在排序数组中出现的次数。例如输入排序数组 {1,2,3,3,3,3,4,5}和数字3，由于3在这个数组出现了4次，因此输出4。

# 分析
对排序的数组使用二分查找，主要为了确定目标数字k的第一个位置和最后一个位置。
在一次查找过程中，存在3种情况：
1. 如果中间数字比k小，则在数组的后半段进行查找；
2. 如果中间数字比k大，则在数组的前半段进行查找；
3. 如果中间数字和k相等，则需要判断这个数字是不是第一个k或者是不是最后一个k。

时间复杂度是$O(\log{n})$。

# 实现
```java
public class NumberOfKInOrderArray {

    public int handle(int[] data, int k) {
        if (data == null || data.length == 0)
            return 0;
        int first = getFirstK(data, k, 0, data.length - 1);
        int last = getLastK(data, k, 0, data.length - 1);

        int num = 0;
        if (first > -1 && last > -1) {
            num = last - first + 1;
        }
        return num;
    }

    private int getFirstK(int[] data, int k, int start, int end) {
        if (start > end)
            return -1;

        int mid = (start + end) >> 1;
        if (data[mid] == k) {
            if ((mid > 0 && data[mid - 1] != k) || mid == 0) {
                return mid;
            } else {
                end = mid - 1;
            }
        } else if (data[mid] < k) {
            start = mid + 1;
        } else {
            end = mid - 1;
        }
        return getFirstK(data, k, start, end);
    }

    private int getLastK(int[] data, int k, int start, int end) {
        if (start > end)
            return -1;

        int mid = (start + end) >> 1;
        if (data[mid] == k) {
            if ((mid < data.length - 1 && data[mid + 1] != k) || mid == data.length - 1) {
                return mid;
            } else {
                start = mid + 1;
            }
        } else if (data[mid] < k) {
            start = mid + 1;
        } else {
            end = mid - 1;
        }
        return getLastK(data, k, start, end);
    }
}
```
