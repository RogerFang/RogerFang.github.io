---
title: 'SwordPointToOffer(8): 旋转数组的最小数字'
date: 2017-01-26 10:41:02
categories:
- SwordPointToOffer
---

# 题目
把一个数组最开始的若干个元素搬到数组的末尾，我们称之为数组的旋转。
输入一个非递减排序的数组的一个旋转，输出旋转数组的最小元素。
例如数组{3,4,5,1,2}为{1,2,3,4,5}的一个旋转，该数组的最小值为1。

# 分析
Bad：直接使用顺序查找，时间复杂度是 O(n)，但是没有利用旋转数组的特性。

和二分查找一样，使用两个指针，分别指向数组的第一个元素和最后一个元素。根据题目中旋转的规则，第一个元素应该是大于或者等于最后一个元素的（不过还存在特例）。

**步骤**：
1. 根据index1和index2求出indexMid（中间元素的索引）
2. 比较indexMid和index1、index2的元素大小
	2.1. 如果`array[indexMid] >= array[index1]`，表示最小元素在indexMid和index2之间；
	2.2. 如果`array[indexMid] < array[index2]`，表示最小元素在index1和indexMid之间。
3. 结束条件：直到index1指向前半部分子数组的最后一个元素，index2指向后半部分子数组的第一个元素。（即index1和index2相邻）

**特例**：
1. 当旋转0个元素时，也就是保持原数组本身。最小元素就是第一个元素。
2. 当index1、index2和indexMid的元素都相等时，没法判断最小元素在哪个区间，因而只能采用顺序查找的办法。

# 实现
```java
public class MinForRotateArray {
    public int minNumberInRotateArray(int [] array) {
        if(array == null || array.length <= 0){
            throw new RuntimeException("invalid input");
        }

        int index1 = 0;
        int index2 = array.length - 1;

        // 旋转0个元素到最后面，即还是排序数组本身
        if(array[index1] < array[index2]){
            return array[index1];
        }

        while(index1 != index2 - 1){
            int indexMid = (index2 + index1)/2;

            // 当索引1和索引2以及中间元素相等时，无法判断最小元素在哪部分子数组，需要使用顺序查找
            if(array[indexMid] == array[index1] && array[indexMid] == array[index2]){
                return minInOrder(array, index1, index2);
            }

            if(array[indexMid] >= array[index1]){
                index1 = indexMid;
            }else{
                index2 = indexMid;
            }
        }
        return array[index2];
    }

    private int minInOrder(int[] array, int start, int end){
        int result = array[start];
        for(int i = start+1; i <= end; ++i){
            if(result > array[i]){
                result = array[i];
            }
        }
        return result;
    }
}
```
