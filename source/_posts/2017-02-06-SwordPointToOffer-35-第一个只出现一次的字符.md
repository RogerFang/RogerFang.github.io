---
title: 'SwordPointToOffer(35): 第一个只出现一次的字符'
date: 2017-02-06 16:38:04
categories:
- SwordPointToOffer
---

# 题目
在字符串中找出第一个只出现一次的字符。如输入“abaccdeff”，则输出“b”。

# 分析
字符是长度为8的数据类型，一共有256种可能，可以用一个数组模拟哈希表记录统计的次数。

# 实现
```java
public class FirstNotRepeatingChar {
    
    public int handle(String str){
        if (str == null || str.length() <= 0)
            return -1;
        
        char[] chars = str.toCharArray();
        int[] hashTable = new int[256];
        
        for (char c: chars){
            hashTable[c]++;
        }
        
        for (int i = 0; i < chars.length; i++){
            if (hashTable[chars[i]] == 1)
                return i;
        }
        return -1;
    }
}
```