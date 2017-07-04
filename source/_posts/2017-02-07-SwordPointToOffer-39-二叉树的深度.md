---
title: 'SwordPointToOffer(39): 二叉树的深度'
date: 2017-02-07 12:42:40
categories:
- SwordPointToOffer
---

# 题目
输入一棵二叉树的根节点，求该树的深度。从根节点到叶节点依次经过的节点（含根、叶节点）形成一条路径，最长路径的长度为树的深度。

# 分析
递归的思想，取左子树和右子树最大的深度加1即为根的深度。

# 实现
```java
public class TreeDepth1 {

    class TreeNode {
        int val = 0;
        TreeNode left = null;
        TreeNode right = null;

        public TreeNode(int val) {
            this.val = val;

        }
    }
    
    public int handle(TreeNode root){
        if (root == null)
            return 0;
        int left = handle(root.left);
        int right = handle(root.right);
        
        return (left > right)?left+1:right+1;
    }
}
```