---
title: 'SwordPointToOffer(34): 丑数'
date: 2017-02-06 16:02:39
categories:
- SwordPointToOffer
---

# 题目
我们把只包含因子2、3和5的数称作丑数。求按从小到大的顺序的第1500个丑数。例如6、8都是丑数，但14不是，习惯上把1称为第一个丑数。

# 分析
## 方法一
逐个判断每个整数是不是丑数，直观但不够高效。
```java
boolean isUgly(int num){
	while(num%2 == 0)
    	num /= 2;
	while(num%3 == 0)
    	num /= 3;
	while(num%5 == 0)
    	num /= 5;
	return num == 1?true:false;
}
```

## 方法二
保存已经找到丑数，用空间换时间的解法。

根据丑数的定义， 丑数应该是另一个丑数乘以 2、3 或者 5 的结果（1 除外）。因此我们可以创建一个数组，里面的数字是排好序的丑数，每一个丑数都是前面的丑数乘以 2、3 或者 5 得到的。

把已有的每个丑数分别都乘以 2、3 和 5，事实上这不是必须的。因为已有的丑数是按顺序存放在数组中的。对乘以 2 而言， 肯定存在某一个丑数 T2，排在它之前的每一个丑数乘以 2 得到的结果都会小于已有最大的丑数，在它之后的每一个丑数乘以 2 得到的结果都会太大。我们只需**记下这个丑数的位置**， 同时每次生成新的丑数的时候，去更新这个 T2。对乘以 3 和 5 而言， 也存在着同样的 T3 和 T5。

# 实现
```java 方法二
import java.util.ArrayList;

public class UglyNumber {

    public int handle(int index){
        ArrayList<Integer> list = new ArrayList<>();
        list.add(1);
        int t2 = 0;
        int t3 = 0;
        int t5 = 0;
        while (list.size() < index){
            int m2 = list.get(t2) * 2;
            int m3 = list.get(t3) * 3;
            int m5 = list.get(t5) * 5;
            int min  = min(m2, m3, m5);
            list.add(min);
            if (m2 == min) t2++;
            if (m3 == min) t3++;
            if (m5 == min) t5++;
        }
        return list.get(index - 1);
    }
    
    private int min(int m2, int m3, int m5){
        return Math.min(m2, Math.min(m3, m5));
    }
}
```