---
title: 'SwordPointToOffer(41): 和为s的两个数字VS和为s的连续整数序列'
date: 2017-02-07 15:30:45
categories:
- SwordPointToOffer
---

# 题目
输入一个递增排序的数组和一个数字s，在数组中查找两个数，使得它们的和正好是s。如果有多对数字的和等于s，输出乘积最小的一对。

例如：输入数组 {1,2,4,7,11,15} 和数字 15，会输出4和11。

# 分析
我们先在数组中选择两个数字 （比如第一个和最后一个），如果它们的和等于输入的 s，我们就找到了要找的两个数字。如果和小于 s 呢？我们希望两个数字的和再大一点。由于数组已经排好序了，我们可以考虑选择较小的数字后面的数字。因为排在后面的数字要大一些，那么两个数字的和也要大一些， 就有可能等于输入的数字 s 了。同样， 当两个数字的和大于输入的数字的时候，我们可以选择较大数字前面的数字，因为排在数组前面的数字要小一些。

关于乘积最小的一对，对于从两头开始夹逼，且是递增数组，如果存在多对情况，即`a<b<c<d`且`a+d = b+c`，这是a和d的乘积就是最小的。（不等式很容易证明）

# 实现
```java
public class TwoNumbersWithSum {

    public ArrayList<Integer> handle(int [] array, int sum) {
        ArrayList<Integer> list = new ArrayList<>();
        if(array == null || array.length <= 1)
            return list;

        int i = 0;
        int j = array.length - 1;
        while(i < j){
            int tmpSum = array[i] + array[j];
            if(tmpSum == sum){
                list.add(array[i]);
                list.add(array[j]);
                return list;
            }else if(tmpSum < sum){
                ++i;
            }else{
                --j;
            }
        }

        return list;
    }
}
```
