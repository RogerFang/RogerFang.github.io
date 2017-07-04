---
title: 'SwordPointToOffer(14): 调整数组顺序使奇数位于偶数前面'
date: 2017-01-29 12:47:05
categories:
- SwordPointToOffer
---

# 题目
输入一个整数数组，实现一个函数来调整该数组中数字的顺序使得所有奇数位于数组的前半部分，所有偶数位于数组的后半部分。

# 分析
时间复杂度$O(n_2)$的思路：从前往后遍历，每遇到一个偶数时，拿出这个数字，并把位于该数字后面的所有数字往前移动一位，再将该数字插入到末尾的空位。

时间复杂度$O(n)$的思路：使用两个指针，一个指向第一位index1，只向后移动；另一个指向最后一位index2，只向前移动。如果index1指向偶数，index2指向奇数则交换这两个数。

# 实现
```java
public void reorder(int[] num) {
    if (num == null) {
        return;
    }

    int index1 = 0;
    int index2 = num.length - 1;

    while (index1 < index2) {
        // 从前往后找到第一个偶数
        while ((num[index1] & 1) != 0) {
            index1++;
        }

        // 从后往前找到第一个奇数
        while ((num[index2] & 1) == 0) {
            index2--;
        }

        if (index1 < index2) {
            int tmp = num[index1];
            num[index1] = num[index2];
            num[index2] = tmp;
        }
    }
}
```