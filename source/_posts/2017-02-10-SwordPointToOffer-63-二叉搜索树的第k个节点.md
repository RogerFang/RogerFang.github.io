---
title: 'SwordPointToOffer(63): 二叉搜索树的第k个节点'
date: 2017-02-10 13:41:05
categories:
- SwordPointToOffer
---

# 题目
给定一棵二叉搜索树，请找出其中的第k个的节点（按节点的数值大小顺序）。

# 分析
如果按照中序遍历的顺序遍历一棵二叉搜索树，遍历序列的数值是递增排序的。

# 实现
```java
public class KthNodeInBST {

    class TreeNode {
        int val = 0;
        TreeNode left = null;
        TreeNode right = null;

        public TreeNode(int val) {
            this.val = val;
        }
    }
    
    public TreeNode findKth(TreeNode root, int k){
        if (root == null || k == 0)
            return null;
        count = k;
        return doFindKth(root);
    }
    
    private int count;
    
    private TreeNode doFindKth(TreeNode root){
        TreeNode target = null;
        
        if (root.left != null)
            target = doFindKth(root.left);
        
        if (target == null){
            if (count == 1)
                target = root;
            else
                count--;
        }
        
        if (target == null && root.right != null)
            target = doFindKth(root.right);
        return target;
    }
}
```