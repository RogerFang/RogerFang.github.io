---
title: 'SwordPointToOffer(28): 字符串的排列'
date: 2017-02-05 15:39:22
categories:
- SwordPointToOffer
---

# 题目
输入一个字符串，打印出该字符串中字符的所有排列。例如输入字符串abc，则打印出由字符a、b、c所能排列出来的所有字符串 abc、acb、bac、bca、cab和cba。（可能有字符重复，字符只包括大小写字母）

# 分析
把一个字符串看成由两部分组成：第一部分为它的第一个字符，第二部分是后面的所有字符。

我们求整个字符串的排列，可以看成两步：首先求所有可能出现在第一个位置的字符，即把第一个字符和后面所有的字符交换。首先固定第一个字符，求后面所有字符的排列。这个时候我们仍把后面的所有字符分成两部分：后面字符的第一个字符，以及这个字符之后的所有字符。然后把第一个字符逐一和它后面的字符交换。

> 注意：
1. 由于可能有字符重复，所以交换时需要做判断，一样的字符不做交换，一定程度上起到了去重减少递归的效果，但是不能达到绝对的去重，因此还是使用`TreeSet`进行存储结果实现去重。
2. 同时在每次递归调用之后需要将前一次的交换操作还原。

# 实现
```java
import java.util.ArrayList;

public class StringPermutation {

    public static void main(String[] args) {
        System.out.println(permutation("cba"));
        System.out.println("******************");
        System.out.println(permutation("abab"));
    }

    public static ArrayList<String> permutation(String str) {
        ArrayList<String> list = new ArrayList<>();
        if(str != null){
            TreeSet<String> set = new TreeSet<>();
            doPermutation(set, str.toCharArray(), 0);
            list.addAll(set);
        }
        return list;
    }
    private static void doPermutation(TreeSet<String> set, char[] chars, int begin){
        if(chars.length - 1 == begin){
            set.add(new String(chars));
        }else{
            for(int i = begin; i < chars.length; i++){
                if (i == begin || chars[i] != chars[begin]){
                    swap(chars, i, begin);
                    doPermutation(set, chars, begin + 1);
                    swap(chars, i, begin);
                }
            }
        }
    }

    private static void swap(char[] chars, int i, int j){
        char tmp = chars[i];
        chars[i] = chars[j];
        chars[j] = tmp;
    }
}
```