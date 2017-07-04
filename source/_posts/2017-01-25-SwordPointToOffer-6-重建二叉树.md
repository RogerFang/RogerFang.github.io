---
title: 'SwordPointToOffer(6): 重建二叉树'
date: 2017-01-25 20:16:49
categories:
- SwordPointToOffer
---

# 题目
输入某二叉树的前序遍历和中序遍历的结果，请重建出该二叉树。假设输入的前序遍历和中序遍历的结果中都不含重复的数字。例如输入前序遍历序列{1,2,4,7,3,5,6,8}和中序遍历序列{4,7,2,1,5,3,8,6}，则重建二叉树并返回。

# 分析
二叉树的前序序列中，第一个数字总是数的根节点的值。但是在中序序列中，根节点的值在序列的中间，左子树的节点值位于根节点的值的左边，而右子树的节点的值位于根节点的值的右边。

先根据前序序列得到根节点的值，然后遍历中序序列得到该值所在的索引，从而分开左子树和右子树。此过程进行递归。

# 实现
```java
public TreeNode reConstructBinaryTree(int[] pre, int[] in) {
    if (pre == null || in == null || pre.length != in.length || pre.length < 1)
        return null;

    return usingRecursion(pre, in, 0, pre.length, 0, in.length);
}

TreeNode usingRecursion(int[] pre, int[] in, int beginP, int endP, int beginI, int endI) {
    int rootVal = pre[beginP];
    TreeNode root = new TreeNode(rootVal);

    int indexOfIn = beginI;
    while (indexOfIn < endI && in[indexOfIn] != rootVal)
        ++indexOfIn;

    if (indexOfIn >= endI) {
        throw new RuntimeException("Invalid input");
    }

    int leftLength = indexOfIn - beginI;
    int rightLength = endI - 1 - indexOfIn;
    if (leftLength > 0) {
        root.left = usingRecursion(pre, in, beginP + 1, beginP + 1 + leftLength, beginI, indexOfIn);
    }
    if (rightLength > 0) {
        root.right = usingRecursion(pre, in, endP - rightLength, endP, indexOfIn + 1, endI);
    }
    return root;
}

class TreeNode {
    int val;
    TreeNode left;
    TreeNode right;

    public TreeNode(int val) {
        this.val = val;
    }
}
```
