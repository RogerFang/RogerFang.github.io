---
title: 'SwordPointToOffer(33): 把数组排成最小的数'
date: 2017-02-06 15:21:33
categories:
- SwordPointToOffer
---

# 题目
输入一个正整数数组，把数组里所有数字都拼接起来排成一个数，打印能拼接处的所有数字中最小的一个。例如输入数组{3,32,321}，则打印出能排成的最小数字321323。

# 分析
## 方法一
求出所有数字的全排列，然后拼接起来，最后求出最大值。n个数字总共有 $n!$个排列。

## 方法二
找到一个排序规则，数组根据这个规则排序之后能排成一个最小的数字。要确定排序规则，就要比较两个数字，也就是给出两个数字 m 和 n，我们需要确定一个规则判断 m 和 n 哪个应该排在前面，而不是仅仅比较这两个数字的值哪个更大。

两个数a和b，排序规则如下：
* 若ab > ba 则 a > b，
* 若ab < ba 则 a < b，
* 若ab = ba 则 a = b；

# 实现
```java
import java.util.Arrays;
import java.util.Comparator;

public class SortArrayForMinNumber {
    
    public String handle(int[] data){
        if (data == null || data.length == 0)
            return null;
        String[] strings = new String[data.length];
        for (int i = 0; i < data.length; ++i){
            if (data[i] <= 0)
                throw new RuntimeException("invalid input!");
            strings[i] = String.valueOf(data[i]);
        }

        Arrays.sort(strings, new Comparator<String>() {
            @Override
            public int compare(String o1, String o2) {
                String s1 = o1 + o2;
                String s2 = o2 + o1;
                return s1.compareTo(s2);
            }
        });

        StringBuilder sb = new StringBuilder();
        for (String s: strings){
            sb.append(s);
        }
        return sb.toString();
    }
}
```