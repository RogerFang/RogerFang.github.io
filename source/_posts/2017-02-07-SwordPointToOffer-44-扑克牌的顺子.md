---
title: 'SwordPointToOffer(44): 扑克牌的顺子'
date: 2017-02-07 20:28:21
categories:
- SwordPointToOffer
---

# 题目
从扑克牌中随机抽5张牌，判断是不是一个顺子，即这5张牌是不是连续的。2~10位数字本身，A为1，J为11，Q为12，K为13，大小王看成任意数字（百搭牌）。

# 分析
我们可以把 5 张牌看成由 5 个数字组成的数组。大、小王是特殊的数字，我们不妨把它们都定义为 0，这样就能和其他扑克牌区分开来了。

接下来我们分析怎样判断 5 个数字是不是连续的，最直观的方法是把数组排序。值得注意的是，由于 0 可以当成任意数字，我们可以用 0 去补满数组中的空缺。如果排序之后的数组不是连续的，即相邻的两个数字相隔若干个数字，但只要我们有足够的。可以补满这两个数字的空缺，这个数组实际上还是连续的。举个例子，数组排序之后为{0，1，3，4，5}在 1 和 3 之间空缺了一个 2，刚好我们有一个 0，也就是我们可以把它当成 2 去填补这个空缺。

于是我们需要做 3 件事情：
1. 首先把数组排序
2. 再统计数组中 0 的个数
3. 最后统计排序之后的数组中相邻数字之间的空缺总数。
如果空缺的总数小于或者等于 0 的个数，那么这个数组就是连续的：反之则不连续。

我们还需要注意一点： 如果数组中的非 0 数字重复出现，则该数组不是连续的。换成扑克牌的描述方式就是如果一副牌里含有对子，则不可能是顺子。

# 实现
```java
import java.util.Arrays;

public class ContinuousCards {
    
    public boolean handle(int[] data){
        if (data == null || data.length < 1)
            return false;
        Arrays.sort(data);
        int numOfZero = 0;
        int numOfGap = 0;
        
        // 统计0的个数
        for (int i: data){
            if (i == 0)
                numOfZero++;
        }
        
        // 统计数组中的间隔数目，对0之后的数字 互相做减法
        int small = numOfZero;
        int big = small + 1;
        while (big < data.length){
            // 两个数相等，有对子
            if (data[small] == data[big])
                return false;
            numOfGap += data[big] - data[small] - 1;
            small = big;
            ++big;
        }
        return numOfGap <= numOfZero;
    }
}
```