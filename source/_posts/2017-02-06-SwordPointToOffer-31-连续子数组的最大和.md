---
title: 'SwordPointToOffer(31): 连续子数组的最大和'
date: 2017-02-06 12:01:00
categories:
- SwordPointToOffer
---

# 题目
输入一个整型数组，数组里有正数也有负数。数组中一个或连续的多个正数组成一个子数组。求所有子数组的和的最大值。要求时间复杂度为O(n)。

例如：输入的数组为{1,-2,3,10,-4,7,2,-5}，和最大的子数组为{3,10,-4,7,2}，因此输出为该子数组的和18。

# 分析
如果使用枚举所有子数组进行求和比较，长度为n的数组由 n(n+1)/2 个子数组，因而时间复杂度是$O(n^2)$。

## 方法
**举例分析数组的规律**

试着从头到尾逐个累加示例数组中的每个数字。初始化和为 0。第一步加上第一个数字 1， 此时和为 1。接下来第二步加上数字 -2，和就变成了 -1。第三步刷上数字3。我们注意到由于此前累计的和是 －1 ，小于 0，那如果用-1 加上 3 ，得到的和是 2 ， 比 3 本身还小。也就是说从第一个数字开始的子数组的和会小于从第三个数字开始的子数组的和。因此我们不用考虑从第一个数字开始的子数组，之前累计的和也被抛弃。

从第三个数字重新开始累加，此时得到的和是 3。接下来第四步加 10，得到和为 13 。第五步加上 -4， 和为 9。我们发现由于 -4 是一个负数，因此累加 -4 之后得到的和比原来的和还要小。因此我们要把之前得到的和 13 保存下来，它有可能是最大的子数组的和。第六步加上数字 7，9 加 7 的结果是 16，此时和比之前最大的和 13 还要大， 把最大的子数组的和由 13 更新为 16。第七步加上 2，累加得到的和为 18，同时我们也要更新最大子数组的和。第八步加上最后一个数字 -5，由于得到的和为 13 ，小于此前最大的和 18，因此最终最大的子数组的和为 18 ，对应的子数组是｛3, 10, -4, 7, 2｝。

| 步骤 | 操作 | 累加的子数组和 | 最大的子数组和 |
|--------|--------|--------|--------------|
|1|加1|1|1|
|2|加-2|-1|1|
|3|抛弃前面的和-1，加3|3|3|
|4|加10|13|13|
|5|加-4|9|13|
|6|加7|16|16|
|7|加2|18|18|
|8|加-5|13|18|

# 实现
```java
public class GreatestSumOfSubarrays1 {

    public static void main(String[] args) {
        int[] data1 = new int[]{1,-2,3,10,-4,7,2,-5};
        int[] data2 = new int[]{-5,-2,-1,-6,-9};
        
        System.out.println(handle(data1));
        System.out.println(handle(data2));

        // System.out.println(0x7fffffff);
        // System.out.println(0x80000000);
    }
    
    public static int handle(int[] data){
        if (data == null || data.length <= 0)
            throw new RuntimeException("invalid input!");
        
        int cumulativeSum = 0;
        int greatestSum = 0x80000000;
        for (int i = 0; i < data.length; i++){
            if (cumulativeSum <= 0){
                cumulativeSum = data[i];
            }else {
                cumulativeSum += data[i];
            }
            
            if (cumulativeSum > greatestSum){
                greatestSum = cumulativeSum;
            }
        }
        return greatestSum;
    }
}
```