---
title: 'SwordPointToOffer(49): 把字符串转换成整数'
date: 2017-02-09 11:45:41
categories:
- SwordPointToOffer
---

# 题目
将一个字符串转换成一个整数，要求不能使用字符串转换整数的库函数。 数值为0或者字符串不是一个合法的数值则返回0。

# 分析
这个题目需要考虑边界条件，非法输入等，当出现非法输入时返回 0，同时还应该设置全局变量`isValid`这样可以和输入正确的字符串 “0”区分开。

# 实现
```java
public class StrToInt {
    
    public boolean isValid = true;
    
    public int handle(String str){
        if (str == null || str.length() <= 0) {
            isValid = false;
            return 0;
        }
        
        char first = str.charAt(0);
        if (first == '-'){
            return parse(str, 1, false);
        }else if (first == '+'){
            return parse(str, 1, true);
        }else if (isDigit(first)){
            return parse(str, 0, true);
        }else {
            isValid = false;
            return 0;
        }
    }
    
    private int parse(String str, int index, boolean positive){
        long tmp = 0;
        while (index < str.length()){
            if (isDigit(str.charAt(index))){
                tmp = tmp * 10 + str.charAt(index) - '0';
                
                if ((positive && tmp > 0x7fffffff) || (!positive && tmp < 0x80000000))
                    throw new NumberFormatException("超过int最大值");
            }else {
                isValid = false;
                return 0;
            }
            index++;
        }
        
        if (positive){
            return (int) tmp;
        }else {
            return (int) -tmp;
        }
    }
    
    private boolean isDigit(char c){
        return c >= '0' && c <= '9';
    }
}
```