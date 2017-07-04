---
title: 'SwordPointToOffer(42): 翻转单词顺序 VS 左旋转字符串'
date: 2017-02-07 16:32:14
categories:
- SwordPointToOffer
---

# 题目
输入一个英文句子，翻转句子中单词的顺序，但单词内字符的顺序不变。为简单起见，标点符号和普通字母一样处理。

例如输入字符串“I am a student.”，则输出“student. a am I”。

# 分析
通过两次翻转实现。

第一步翻转句子中所有的字符。比如翻转“I am a student. ”中所有的字符得到”.tneduts a m a I”，此时不但翻转了句子中单词的顺序，连单词内的字符顺序也被翻转了。第二步再翻转每个单词中字符的顺序，就得到了”student. a am I”。这正是符合题目要求的输出。

# 实现
```java
public class ReverseWordsInSentence {
    
    public String handle(String str){
        if (str == null || str.length() <= 0)
            return null;
        
        char[] data = str.toCharArray();
        // 翻转整个句子
        reverse(data, 0, data.length-1);
        // 翻转句子中的每个单词
        int start = 0;
        int end = 0;
        while (start < data.length){
            if (data[start] == ' '){
                ++start;
                ++end;
            }else if (end == data.length || data[end] == ' '){
                reverse(data, start, end - 1);
                ++end;
                start = end;
            }else {
                ++end;
            }
        }
        return new String(data);
    }
    
    private void reverse(char[] data, int start, int end){
        if (data == null || start >= end)
            return;
        while (start < end){
            char tmp = data[start];
            data[start] = data[end];
            data[end] = tmp;
            
            ++start;
            --end;
        }
    }
}
```