---
title: 'SwordPointToOffer(32): 从1到n整数中1出现的次数'
date: 2017-02-06 13:22:15
categories:
- SwordPointToOffer
---

#题目
输入一个整数n，求从1到n这n个整数的十进制表示中1出现的次数。例如输入12，从1到12这些整数中国包含1的数字有1,10,11和12，1一共出现了5次。

# 分析
如果直接累加1到n中每个整数出现1的次数，采用做除法和求余运算以求出该数字中1出现的次数。如果输入数字n，n有$O(\log{n})$位，因此该算法的时间复杂度是$O(n\log{n})$。运算效率不高。
## 方法
**从数字规律着手明显提高时间效率的解法**。

**数字的规律**：
从右往左，给每位标号1,2,3...，当统计x的次数时，需要分两种情况讨论：$x\in[1,9]$和 $x=0$。


### $x\in[1,9]$情况
第`i`位上的数字记为`t`时，高位记为`a`，低位记为`b`，因此一个整数可以记为`atb`形式。
分下面三种情况：
1. 当`t < x`时，高位可取`[0, a-1]`，第i位的计数为 $a\times10^{i-1}$。
2. 当`t = x`时，高位可取`[0, a]`，此时还受低位的影响，第i位的计数为 $a\times10^{i-1} + (b+1)$。
3. 当`t > x`时，高位可取`[0, a]`，此时不受低位的影响，第i位的计数为 $(a+1)\times10^{i-1}$。

### $x=0$情况
第`i`位上的数字记为`t`时，高位记为`a`，低位记为`b`，因此一个整数可以记为`atb`形式。
分下面两种种情况：
1. 当`t = 0`时，高位可取`[1, a]`，此时还受低位的影响，第i位的计数为 $(a-1)\times10^{i-1} + (b+1)$。
2. 当`t > 0`时，高位可取`[1, a]`，此时不受低位的影响，第i位的计数为 $a\times10^{i-1}$。

# 实现
```java
/**
 * 求出1到n整数中出现x(1<=x<=9)的次数
 */
public class NumberOfX {

    public static void main(String[] args) {
        System.out.println(handle(13, 1));
    }
    
    public static int handle(int num, int x) {
        if (num < 1)
            throw new RuntimeException("invalid input!");

        int sum = 0;
        int high = num;
        int low, cur, tmp;
        int i = 1;
        while (high != 0) {
            high = num / (int) Math.pow(10, i);
            tmp = num % (int) Math.pow(10, i);
            cur = tmp / (int) Math.pow(10, i - 1);
            low = tmp % (int) Math.pow(10, i - 1);

            if (cur == x) {
                sum += high * (int) Math.pow(10, i - 1) + low + 1;
            } else if (cur < x) {
                sum += high * (int) Math.pow(10, i - 1);
            } else {
                sum += (high + 1) * (int) Math.pow(10, i - 1);
            }
            ++i;
        }
        return sum;
    }
}
```