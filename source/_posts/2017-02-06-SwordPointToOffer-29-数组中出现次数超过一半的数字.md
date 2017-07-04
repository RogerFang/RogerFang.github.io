---
title: 'SwordPointToOffer(29): 数组中出现次数超过一半的数字'
date: 2017-02-06 10:09:20
categories:
- SwordPointToOffer
---

# 题目
数组中有一个数字出现的次数超过数组长度的一半，请找出这个数字。例如输入一个数组 {1,2,3,2,2,2,2,5,4,2}，会输出数字2。

# 分析
## 方法一
> 时间复杂度最优是$O(n\log{n})$。

将整个数组进行排序，取出`n/2`索引处的数字即可。

## 方法二
> 时间复杂度是$O(n)$。

借用快速排序的`partition`函数，简化整个方法一的整个排序过程。随机选择一个基数索引`pivotIndex`，如果`pivotIndex > middle`就在基数索引的左边进行查找；如果`pivotIndex < middle`就在基数索引的右边进行查找。

**这种方法会修改输入的数组**。

## 方法三
> 时间复杂度是$O(n)$。

数组中有一个数字出现的次数超过数组长度的一半，也就是说它出现的次数比其他所有数字出现次数的和还要多。因此我们可以考虑在遍历数组的时候保存两个值： 一个是数组中的一个数字， 一个是次数。当我们遍历到下～个数字的时候，如果下一个数字和我们之前保存的数字相同，则次数加 l ：如果下一个数字和我们之前保存的数字，不同，则次数减 1 。如果次数为零，我们需要保存下一个数字，并把次数设为 1 。由于我们要找的数字出现的次数比其他所有数字出现的次数之和还要多，那么要找的数字肯定是最后一次把次数设为 1 时对应的数字。

# 实现
```java 方法二partition函数
/**
 * 基于快速排序的partition
 */
public class MoreThanHalfNumber1 {

    public static void main(String[] args) {
        MoreThanHalfNumber1 test = new MoreThanHalfNumber1();
        int[] data = new int[]{1,2,3,2,2,2,1};
        System.out.println(test.handle(data));
    }
    
    public int handle(int[] data){
        if (data == null || data.length <= 0)
            throw new RuntimeException("invalid input!");
        
        int middle = data.length >> 1;
        int start = 0;
        int end = data.length - 1;
        int index = partition(data, start, end, end);
        while (index != middle){
            if (index < middle){
                start = index + 1;
                index = partition(data, start, end, end);
            }else {
                end = index - 1;
                index = partition(data, start, end, end);
            }
        }
        
        int result = data[middle];
        if (!checkMoreThanHalf(data,result))
            throw new RuntimeException("there is no number more than half Num");
        return result;
    }
    
    private boolean checkMoreThanHalf(int[] data, int result){
        int times = 0;
        for(int i = 0; i < data.length; ++i){
            if (data[i] == result){
                ++times;
            }
        }
        return times * 2 > data.length;
    }

    private int partition(int[] data, int start, int end, int pivotIndex){
        int small = start;
        int large = end;
        int pivot = data[pivotIndex];
        swap(data, end, pivotIndex);
        while (small != large){
            while (data[small] <= pivot && small < large)
                small++;
            while (data[large] >= pivot && small < large)
                large--;

            if (small < large)
                swap(data, small, large);
        }

        // 基数归位
        swap(data, end, small);
        return small;
    }

    private void swap(int[] data, int i, int j){
        int tmp = data[i];
        data[i] = data[j];
        data[j] = tmp;
    }
}
```


```java 方法三
public class MoreThanHalfNumber2 {

    public static void main(String[] args) {
        MoreThanHalfNumber2 test = new MoreThanHalfNumber2();
        int[] data = new int[]{1,2,3,2,2,1,1};
        System.out.println(test.handle(data));
    }
    
    public int handle(int[] data){
        if (data == null || data.length <= 0)
            throw new RuntimeException("invalid input!");

        int result = data[0];
        int times = 1;
        for (int i = 1; i < data.length; ++i){
            if (times == 0){
                result = data[i];
                times = 1;
            }else if (data[i] == result){
                times++;
            }else {
                times--;
            }
        }

        if (!checkMoreThanHalf(data, result))
            throw new RuntimeException("there is no number more than half Num");
        return result;
    }

    private boolean checkMoreThanHalf(int[] data, int result){
        int times = 0;
        for(int i = 0; i < data.length; ++i){
            if (data[i] == result){
                ++times;
            }
        }
        return times * 2 > data.length;
    }
}
```
