---
title: 'SwordPointToOffer(3): 二维数组中的查找'
date: 2017-01-25 12:59:31
categories:
- SwordPointToOffer
---

# 题目
在一个二维数组中，每一行都按照从左到右递增的顺序排序，每一列都按照从上到下递增的顺序排序。请完成一个函数，输入这样的一个二维数组和一个整数，判断数组中是否含有该整数。

# 分析
原数组有序，从左到右递增，从上到下递增，步骤如下：
1. 选取数组中右上角(或者左下角)的数字
2. 和target数字比较
    2.1 如果相等，则返回true
    2.2 如果大于target，则删除列（--column）
    2.3 如果小于target，则删除行（++row）

注意边界检查。
> 选取右上角或者左下角的数字和target数字比较的原因是，每次比较后都能缩小查找的范围，如果选取左上角或者右下角则不能。

# 实现
```java
public class Solution {
    public boolean Find(int target, int [][] array) {
        boolean found = false;

        if(array != null && array.length > 0 && array[0] != null && array[0].length > 0){
            int row = 0;
            int column = array[0].length - 1;
            while(row < array.length && column >= 0){
                int cur = array[row][column];
                if(cur == target){
                    found = true;
                    break;
                }else if(cur > target){
                    --column;
                }else{
                    ++row;
                }
            }
        }

        return found;
    }
}
```

# 测试用例
* 二维数组中包含target数字（target为最大值或最小值，或者是中间数）
* 二维数组中没有target数字
* 特殊输入测试（空指针）