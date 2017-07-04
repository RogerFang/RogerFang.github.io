---
title: 'SwordPointToOffer(43): n个骰子的点数'
date: 2017-02-07 19:13:37
categories:
- SwordPointToOffer
---

# 题目
把n个骰子仍在地上，所有骰子朝上一面的点数之和为s。输入n，打印出s的所有可能的值的概率。

# 分析
## 方法一
> 基于**递归**求解，时间效率不够高，有很多重复的计算。

先把 n 个骰子分为两堆：第一堆只有一个，另一个有 n-1 个。单独的那一个有可能出现从 1 到 6 的点数。我们需要计算从 1 到 6 的每一种点数和剩下的 n-1 个骰子来计算点数和。接下来把剩下的 n-1 个骰子还是分成两堆，第一堆只有一个， 第二堆有 n-2 个。我们把上一轮那个单独骰子的点数和这一轮单独骰子的点数相加， 再和剩下的 n-2 个骰子来计算点数和。分析到这里，我们不难发现这是一种递归的思路，递归结束的条件就是最后只剩下一个骰子。

我们可以定义一个长度为`6n-n+1`的数组， 和为 s 的点数出现的次数保存到数组第`s-n`个元素里。

## 方法二
> 基于循环求解，时间性能好。

我们可以考虑用两个数组来存储骰子点数的每一个总数出现的次数。在一次循环中， 第一个数组中的第 n 个数字表示骰子和为 n 出现的次数。在下一循环中，我们加上一个新的骰子，此时和为 n 的骰子出现的次数应该等于上一次循环中骰子点数和为 n-1 、n-2 、n-3 、n-4, n-5 与 n-6 的次数的总和，所以我们把另一个数组的第 n 个数字设为前一个数组对应的第 n-1 、n-2 、n-3 、n-4、n-5 与 n-6 之和。

# 实现
```java 方法一
/**
 * 递归实现
 */
public class PrintProbRecusive {

    public static void main(String[] args) {
        PrintProbRecusive test = new PrintProbRecusive();
        test.handle(2);
    }

    // 骰子最大的点数
    private static final int MAX_VAL = 6;

    public void handle(int n){
        if (n < 1)
            return;

        int maxSum = n * MAX_VAL;
        int total = (int) Math.pow(MAX_VAL, n);
        int[] countArray = new int[maxSum - n + 1];

        getProbs(n, countArray);
        for (int i = n; i <= maxSum; ++i){
            System.out.printf("%d: %e\n", i, (double)countArray[i-n]/total);
        }
    }

    private void getProbs(int n, int[] countArray){
        for (int i = 1; i <= MAX_VAL; ++i){
            getProbs(n, n, i, countArray);
        }
    }

    private void getProbs(int originalN, int currentN, int sum, int[] countArray){
        if (currentN == 1){
            countArray[sum - originalN]++;
        }else {
            for (int i = 1; i <= MAX_VAL; ++i){
                getProbs(originalN, currentN-1, sum+i, countArray);
            }
        }
    }
}
```


```java 方法二
/**
 * 循环实现
 */
public class PrintProbCycle {

    public static void main(String[] args) {
        PrintProbCycle test = new PrintProbCycle();
        test.handle(2);
    }

    // 骰子最大的点数
    private static final int MAX_VAL = 6;

    public void handle(int n){
        if (n < 1)
            return;
        // 第0位没有用
        int[][] probs = new int[2][MAX_VAL * n + 1];
        // for (int i = 0)
        // 标记当前要使用的是第0个还是第1个数组
        int flag = 0;
        // 抛出一个骰子时的情况
        for (int i = 1; i <= MAX_VAL; ++i){
            probs[flag][i] = 1;
        }
        // 抛出其他骰子
        for (int k = 2; k <= n; ++k){
            // 如果抛出了k个骰子，那么和为[0,k)的出现次数为0
            for (int i = 0; i < k; ++i){
                probs[1-flag][i] = 0;
            }
            // 抛出k个骰子，所有和的可能
            for (int i = k; i <= k*MAX_VAL; ++i){
                for (int j = 1; j < i && j <= MAX_VAL; ++j){
                    // 统计出现和为i的次数
                    probs[1-flag][i] += probs[flag][i-j];
                }
            }
            flag = 1 - flag;
        }

        double total = Math.pow(MAX_VAL, n);
        for (int i = n; i <= MAX_VAL*n; ++i){
            System.out.printf("%d: %e\n", n, (double)probs[flag][i]/total);
        }
    }
}
```