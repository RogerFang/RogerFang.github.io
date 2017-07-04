---
title: 'SwordPointToOffer(19): 二叉树的镜像'
date: 2017-01-29 20:49:47
categories:
- SwordPointToOffer
---

# 题目
请完成一个函数，输入一个二叉树，该函数输出它的镜像。

# 分析
从上往下递归交换左右子树。

# 实现
```java
public class MirrorTree {

    class TreeNode{
        int value;
        TreeNode left;
        TreeNode right;
    }

    public void mirror(TreeNode root) {
        if(root == null){
            return;
        }

        if(root.left == null && root.right == null){
            return;
        }

        TreeNode tmp = root.left;
        root.left = root.right;
        root.right = tmp;

        if(root.left != null){
            mirror(root.left);
        }

        if(root.right != null){
            mirror(root.right);
        }
    }
}
```