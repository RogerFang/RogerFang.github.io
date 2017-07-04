---
title: 'SwordPointToOffer(42): 翻转单词顺序 VS 左旋转字符串(补充)'
date: 2017-02-07 16:57:02
categories:
- SwordPointToOffer
---

# 题目
字符串的左旋转操作时把字符串前面的若干个字符转移到字符串的尾部。请定义一个函数实现字符串左旋转操作的功能。

例如输入字符串“abcdefg”和数字2，该函数将返回左旋转2位得到的结果“cdefgab”。

# 分析
以”abcdefg”为例，我们可以把它分为两部分。由于想把它的前两个字符移到后面，我们就把前两个字符分到第一部分，把后面的所有字符都分到第二部分。我们先分别翻转这两部分，于是就得到”bagfedc”。接下来我们再翻转整个字符串， 得到的”cde 也 ab”刚好就是把原始字符串左旋转 2 位的结果。

只需要调用前面的翻转函数3次即可。

# 实现
```java
public class LeftRotateString {

    public String handle(String str, int n) {
        if (str == null || str.length() < n)
            return "";

        char[] data = str.toCharArray();
        reverse(data, 0, n - 1);
        reverse(data, n, data.length - 1);
        reverse(data, 0, data.length - 1);
        return new String(data);
    }

    private void reverse(char[] data, int start, int end) {
        if (data == null || start >= end)
            return;
        while (start < end) {
            char tmp = data[start];
            data[start] = data[end];
            data[end] = tmp;

            ++start;
            --end;
        }
    }
}
```
