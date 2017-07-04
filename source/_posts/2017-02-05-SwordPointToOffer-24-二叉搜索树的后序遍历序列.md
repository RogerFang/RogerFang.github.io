---
title: 'SwordPointToOffer(24): 二叉搜索树的后序遍历序列'
date: 2017-02-05 11:11:36
categories:
- SwordPointToOffer
---

# 题目
输入一个整数数组，判断该数组是不是某二叉搜索树的后序遍历的结果。如果是则返回true，否则返回false。假设输入的数组的任意两个数字都互不相同。

# 分析
后序遍历的序列中最后一个总是树（子树）的根节点，同时二叉搜索树的性质是左子树都小于根节点，且右子树都大于根节点。确定根节点后，依次遍历其子树序列，找出分界节点。

# 实现
```java
public class VerifySequenceOfBST {

    public static void main(String[] args) {
        int[] sequence1 = new int[]{5, 7, 6, 9, 11, 10, 8};
        int[] sequence2 = new int[]{7, 4, 6, 5};

        System.out.println(verify(sequence1));
        System.out.println(verify(sequence2));
    }

    public static boolean verify(int [] sequence) {
        if(sequence == null || sequence.length == 0){
            return false;
        }
        return doVerify(sequence, 0, sequence.length - 1);
    }

    private static boolean doVerify(int[] sequence, int begin, int end){
        if(begin > end || begin < 0 || end >= sequence.length){
            return false;
        }

        int root = sequence[end];
        int i = begin;
        for(; i < end; i++){
            if(sequence[i] > root)
                break;
        }

        int j = i;
        for(; j < end; j++){
            if(sequence[j] < root)
                return false;
        }

        boolean left = true;
        if(i > 0)
            left = doVerify(sequence, begin, i - 1);

        boolean right = true;
        if(j < end && end > 0)
            right = doVerify(sequence, i, end - 1);

        return left&&right;
    }
}
```