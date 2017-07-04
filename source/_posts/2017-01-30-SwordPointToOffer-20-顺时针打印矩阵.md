---
title: 'SwordPointToOffer(20): 顺时针打印矩阵'
date: 2017-01-30 10:25:10
categories:
- SwordPointToOffer
---

# 题目
输入一个矩阵，按照从外往里以顺时针的顺序依次打印出每一个数字。

# 分析
把打印一圈分为四步：第一步从左到右打印一行，第二步从上到下打印一列，第三步从右到左打印一行，第四步从下到上打印一列。每一步我们根据起始坐标和终止坐标用一个循环就能打印出一行或者一列。

最后一圈有可能退化成只有一行、只有一列，甚至只有一个数字。也可能只有一步、两步、三步。

继续打印圈的终止条件：`startX * 2 < columns && startY * 2 < rows`

# 实现
```java
public class PrintMatrixClockWisely {

    public ArrayList<Integer> printMatrix(int [][] matrix) {
        if(matrix == null || matrix[0].length <= 0){
            return null;
        }

        ArrayList<Integer> list = new ArrayList<>();
        int start = 0;
        while(start*2 < matrix.length && start*2 < matrix[0].length){
            print(list, matrix, start);
            start++;
        }
        return list;
    }

    private void print(ArrayList<Integer> list, int [][] matrix, int start){
        int endX = matrix[0].length - 1- start;
        int endY = matrix.length - 1 - start;
        // 从左到右 打印一行
        for(int i = start; i <= endX; i++){
            list.add(matrix[start][i]);
        }

        // 从上到下 打印一列
        if(start < endY){
            for(int i = start + 1; i <= endY; ++i){
                list.add(matrix[i][endX]);
            }
        }

        // 从右到左 打印一行
        if(start < endX && start < endY){
            for(int i = endX - 1; i >= start; --i){
                list.add(matrix[endY][i]);
            }
        }

        // 从下到上 打印一列
        if(start < endX && start < endY - 1){
            for(int i = endY - 1; i >= start + 1; --i){
                list.add(matrix[i][start]);
            }
        }
    }
}
```