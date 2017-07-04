---
title: 'SwordPointToOffer(30): 最小的k个数'
date: 2017-02-06 10:33:28
categories:
- SwordPointToOffer
---

# 题目
输入n个整数，找出其中最小的k个数。例如输入 4、5、1、6、2、7、3、8，则最小的4个数字是1、2、3、4。

# 分析
## 方法一
> 时间复杂度最优是$O(n\log{n})$。

最简单的方法就是对整个输入的数字进行排序，然后取出前k个数。

## 方法二
> 时间复杂度是$O(n)$。 和29题的方法二一样

借助快速排序的partition函数，随机选择一个基数索引`pivotIndex`，如果`pivotIndex > k`就在基数索引的左边进行查找；如果`pivotIndex < k`就在基数索引的右边进行查找。

**这种方法会修改输入的数组**。

## 方法三
> 时间复杂度是$O(n\log{k})$。

先创建一个大小为 k 的数据容器来存储最小的 k 个数字，接下来我们每次从输入的 n 个整数中读入一个数．如果容器中已有的数字少于 k 个，则直接把这次读入的整数放入容器之中：如果容器中己有k 数字了，也就是容器己满，此时我们不能再插入新的数字而只能替换已有的数字。找出这己有的 k 个数中的最大值，然后 1 在这次待插入的整数和最大值进行比较。如果待插入的值比当前己有的最大值小，则用这个数替换当前已有的最大值：如果待插入的值比当前已有的最大值还要大，那么这个数不可能是最小的k个整数之一，于是我们可以抛弃这个整数。

因此当容器满了之后，我们要做 3 件事情： 一是在 k 个整数中找到最大数： 二是有可能在这个容器中删除最大数： 三是有可能要插入一个新的数字。我们可以使用一个**最大堆**在 $O(\log{k})$时间内实现这三步操作。

**这种方法特别适合处理海量数据**。

# 实现
```java 方法二
import java.util.ArrayList;

/**
 * 基于partition函数
 */
public class KLeastNumbers2 {
    
    public ArrayList<Integer> handle(int[] data, int k){
        if (data == null || k <= 0 || data.length < k)
            throw new RuntimeException("invalid input!");
        
        int start = 0;
        int end = data.length - 1;
        int index = partition(data, start, end, end);
        while (index != k - 1){
            if (index < k - 1){
                start = index + 1;
                index = partition(data, start, end, end);
            }else{
                end = index - 1;
                index = partition(data, start, end, end);
            }
        }
        
        ArrayList<Integer> list = new ArrayList<>();
        for (int i = 0; i < k; ++i){
            list.add(data[i]);
        }
        return list;
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
import java.util.ArrayList;
import java.util.Comparator;
import java.util.PriorityQueue;

/**
 * 使用容器存储k个数
 */
public class KLeastNumbers2 {
    
    public ArrayList<Integer> handle(int[] data, int k){
        if (data == null || k <= 0 || data.length < k)
            throw new RuntimeException("invalid input!");

        PriorityQueue<Integer> queue = new PriorityQueue<>(k, new Comparator<Integer>() {
            @Override
            public int compare(Integer o1, Integer o2) {
                return o2 - o1;
            }
        });
        
        for (int i = 0; i < data.length; ++i){
            if (queue.size() < k){
                queue.offer(data[i]);
            }else {
                if (data[i] < queue.peek()){
                    queue.poll();
                    queue.offer(data[i]);
                }
            }
        }
        
        ArrayList<Integer> list = new ArrayList<>();
        for (Integer i: queue){
            list.add(i);
        }
        
        return list;
    }
}
```