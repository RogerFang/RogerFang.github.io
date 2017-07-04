---
title: 'SwordPointToOffer(53): 正则表达式匹配'
date: 2017-02-09 13:56:39
categories:
- SwordPointToOffer
---

# 题目
请实现一个函数用来匹配包括'.'和'*'的正则表达式。模式中的字符'.'表示任意一个字符，而'*'表示它前面的字符可以出现任意次（包含0次）。 在本题中，匹配是指字符串的所有字符匹配整个模式。例如，字符串"aaa"与模式"a.a"和"ab*ac*a"匹配，但是与"aa.a"和"ab*a"均不匹配

# 分析
每次从字符串里拿出**一个字符**和模式中的字符去匹配。先来分析如何匹配一个字符。如果模式中的字符 ch是‘.’，那么它可以匹配字符串中的任意字符。如果模式中的字符 ch不是’.’而且字符串中的字符也是 ch，那么他们相互匹配。当字符串中的字符和模式中的字符相匹配时，接着匹配后面的字符。

相对而言当模式中的**第二个字符**不是‘*’时问题要简单很多。如果字符串中的第一个字符和模式中的第一个字符相匹配，那么在字符串和模式上都向后移动一个字符，然后匹配剩余的字符串和模式。如果字符串中的第一个字符和模式中的第一个字符不相匹配，则直接返回 false。

当模式中的**第二个字符**是‘*’时问题要复杂一些，因为可能有多种不同的匹配方式。一个选择是在模式上向后移动两个字符。这相当于‘’和它面前的字符被忽略掉了，因为‘*’可以匹配字符串中 0 个字符。如果模式中的第一个字符和字符串中的第一个字符相匹配时，则在字符串向后移动一个字符，而在模式上有两个选择：我们可以在模式上向后移动两个字符，也可以保持模式不变。

# 实现
```java
public class RegularExpressionsMatch {

    public static void main(String[] args) {
        RegularExpressionsMatch test = new RegularExpressionsMatch();
        System.out.println(test.match(new char[]{' '}, new char[]{'.','*'}));
    }

    public boolean match(char[] input, char[] pattern)
    {
        if (input == null || pattern == null) {
            return false;
        }
        return matchCore(input, 0, pattern, 0);
    }

    private boolean matchCore(char[] input, int i, char[] pattern, int p) {
        // 匹配串和模式串都到达尾，说明成功匹配
        if (i >= input.length && p >= pattern.length) {
            return true;
        }
        // 只有模式串到达结尾，说明匹配失败
        if (i < input.length && p >= pattern.length) {
            return false;
        }
        // 模式串未结束, 匹配串有可能结束有可能未结束
        // 第二个字符是'*', p位置的下一个字符中为*号
        if (p + 1 < pattern.length && pattern[p + 1] == '*') {
            // 第一个字符匹配, 第二个字符是'*', 分为3种情况
            if (i < input.length && (input[i] == pattern[p] || pattern[p] == '.')) {
                return matchCore(input, i, pattern, p + 2) ||
                        matchCore(input, i + 1, pattern, p) ||
                        matchCore(input, i + 1, pattern, p + 2);
            }
            // 第一个字符不匹配, 第二个字符是'*', 直接略过'*'往后匹配
            else {
                return matchCore(input, i, pattern, p + 2);
            }
        }

        // 第二个字符不是 '*', 第一个字符匹配
        if (i < input.length && (input[i] == pattern[p] || pattern[p] == '.'))
            return matchCore(input, i + 1, pattern, p + 1);

        return false;
    }
}
```