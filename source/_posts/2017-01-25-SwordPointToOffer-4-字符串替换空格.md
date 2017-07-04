---
title: 'SwordPointToOffer(4): 字符串替换空格'
date: 2017-01-25 14:10:32
categories:
- SwordPointToOffer
---

# 题目
请实现一个函数，将一个字符串中的空格替换成“%20”。例如，当字符串为We Are Happy.则经过替换之后的字符串为We%20Are%20Happy。

# 分析
对于字符串替换，会造成字符串的长度变化，所以有两种方式解决：1.在位的替换字符；2，通过创建新的字符串来完成字符替换操作。

在位解决思路（时间复杂度为**O(n)**）：
1. 先遍历一次字符数组，获取空格的数量，由此计算出替换之后的字符串总长度。
2. 准备两个索引指针`indexOfOriginal`和`indexOfNew`，分别指向原字符串的末尾和新字符串的末尾。
3. 向前移动指针`indexOfOriginal`，逐个把字符复制到`indexOfNew`指向的位置，直到遇到空格符时进行相应的字符替换。

# 在位实现
```java
/**
 * @param string 字符数组
 * @param length 字符串实际大小
 * @return 转换后的字符串实际长度，-1表示失败
 */
public static int replaceBlank(char[] string, int length){
    if (string == null || length <= 0){
        return -1;
    }

    int numOfBlank = 0;
    for (int i = 0; i < length; i++){
        if (string[i] == ' '){
            numOfBlank++;
        }
    }

    if (numOfBlank == 0){
        return length;
    }

    int newLength = length + 2 * numOfBlank;
    if (newLength > string.length){
        return -1;
    }

    int indexOfOriginal = length - 1;
    int indexOfNew = newLength - 1;
    while (indexOfOriginal >= 0 && indexOfNew > indexOfOriginal){
        if (string[indexOfOriginal] == ' '){
            string[indexOfNew--] = '0';
            string[indexOfNew--] = '2';
            string[indexOfNew--] = '%';
        }else {
            string[indexOfNew--] = string[indexOfOriginal];
        }
        --indexOfOriginal;
    }

    return newLength;
}
```