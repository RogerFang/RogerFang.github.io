---
title: 'SwordPointToOffer(12): 打印1到最大的n位数'
date: 2017-01-29 11:07:06
categories:
- SwordPointToOffer
---

# 题目
输入数字n，按顺序打印出从1到最大的n位十进制数。比如输入3，则打印出1,2,3,...,999。

# 分析
问题：需要注意这个题目的陷阱，最大的n位数可能会造成数据类型（int 或 long）的溢出，所以这是一个**大数问题**，应该使用字符串或字符数组来解决。

有两种方法:
首先定义一个长度为n的字符数组，默认所有位置填充字符 '0'。
1. **在字符串上模拟数字的加法**
2. **利用全排列思想的递归实现**

打印数字的时候前面的填充 '0' 注意不打印。

# 实现
```java
/**
 * 打印1到最大的n位数
 * 考虑大数问题
 * Created by Roger on 2017/1/29.
 */
public class PrintMaxN {
    public static void main(String[] args) {
        PrintMaxN test = new PrintMaxN();
        // test.method1(3);
        test.method2(2);
    }

    /**
     * 利用字符串来模拟整数的加法并打印
     * @param n
     */
    public void method1(int n){
        if (n <= 0){
            return;
        }
        
        char[] num = new char[n];
        for(int i = 0; i < n; i++){
            num[i] = '0';
        }
        
        while (!increment(num)){
            print(num);
        }
    }

    /**
     * 把问题转换为数字全排列
     * @param n
     */
    public void method2(int n){
        if (n <= 0){
            return;
        }

        char[] num = new char[n];
        for(int i = 0; i < n; i++){
            num[i] = '0';
        }

        /*for (int i = 0; i < 10; i++){
            num[0] = (char)(i + '0');
            usingRecursion(num, 0);
        }*/
        usingRecursion(num, 0);
    }

    /**
     * 字符串模拟整数的加法
     * @param num
     * @return 是否溢出
     */
    private boolean increment(char[] num){
        boolean isOverFlow = false;
        int nTakeOver = 0;
        int nLength = num.length;
        
        for (int i = nLength - 1; i >= 0; i--){
            int sum = num[i] - '0' + nTakeOver;
            if (i == nLength - 1){
                sum++;
            }
            
            if (sum >= 10){
                if (i == 0){
                    isOverFlow = true;
                }else {
                    sum -= 10;
                    nTakeOver = 1;
                    num[i] = (char)('0' + sum);
                }
            }else {
                num[i] = (char) ('0' + sum);
                break;
            }
        }
        
        return isOverFlow;
    }

    /**
     * 模拟全排列递归实现
     * @param num
     * @param index
     */
    private void usingRecursion(char[] num, int index){
        if (index == num.length){
            print(num);
            return;
        }
        
        for (int i = 0; i < 10; i++){
            num[index] = (char)(i + '0');
            usingRecursion(num, index + 1);
        }
    }
    
    private void print(char[] num){
        boolean isBeginning0 = true;
        for (int i = 0; i < num.length; i++){
            if (isBeginning0 && num[i] != '0'){
                isBeginning0 = false;
            }
            
            if (!isBeginning0){
                System.out.print(num[i]);
            }
        }
        System.out.println();
    }
}
```