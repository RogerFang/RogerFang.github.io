---
title: 'SwordPointToOffer(11): 数值的整数次方'
date: 2017-01-27 21:56:05
categories:
- SwordPointToOffer
---

# 题目
给定一个double类型的浮点数base和int类型的整数exponent。求base的exponent次方。
不得使用库函数，同时不需要考虑大数问题。

# 分析
这个题目看似简单，但是有很多问题需要考虑。
1. 边界问题
	当底数base是0且指数是负数时，需要做特殊处理。
2. 浮点型equal问题
	计算机表示小数（float和double）都有误差，因而不能直接使用等号（==）进行比较。如果两个小数的差的绝对值很小，就可以认为它们相等。
3. 效率问题
	可以不用进行exponent次乘法完成指数计算，比如：求一个树的32次方，如果已经计算得到了16次方，那么只需要在16次方的基础上再平方一次就可以了。

# 实现
```java
public class NumberExponentialOperation {
    public double Power(double base, int exponent) {
        if(equal(base, 0.0) && exponent < 0){
            throw new RuntimeException("invalid input");
        }

        int absExponent = exponent;
        if(exponent < 0){
            absExponent = -exponent;
        }
        double result = doPowerWithExponent(base, absExponent);
        if(exponent < 0){
            result = 1.0/result;
        }
        return result;
    }

    private double doPowerWithExponent(double base, int exponent){
        if(exponent == 0){
            return 1;
        }

        if(exponent == 1){
            return base;
        }

        double result = doPowerWithExponent(base, exponent >> 1);
        result *= result;
        if((exponent & 0x1) == 1){
            result *= base;
        }
        return result;
    }

    private boolean equal(double d1, double d2){
        if((d1 - d2 > -0.0000001) && (d1 - d2 < 0.0000001)){
            return true;
        }else
            return false;
    }
}
```