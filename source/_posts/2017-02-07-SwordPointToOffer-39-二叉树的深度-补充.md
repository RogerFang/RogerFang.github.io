---
title: 'SwordPointToOffer(39): 二叉树的深度(补充)'
date: 2017-02-07 14:00:12
categories:
- SwordPointToOffer
---

# 题目
输入一棵二叉树的根节点，判断该树是不是平衡二叉树。如果某二叉树中任意节点的左右子树的深度相差不超过1，那么它就是一棵平衡二叉树。

# 分析
## 方法一
前一篇涉及到如何求二叉树的深度，借用这个可以递归实现平衡判断。

该方法不足之处在于需要重复遍历节点多次。

## 方法二
用后序遍历的方式遍历二叉树的每一个节点，在遍历到一个节点之前我们就已经遍历了它的左右子树。只要在遍历每个节点的时候记录它的深度（某一节点的深度等于它到叶节点的路径的长度），我们就可以一边遍历一边判断每个节点是不是平衡的。

## 方法三
在后序遍历求二叉树深度的同时，设置一个全局变量，自底向上判断二叉树是否平衡。

# 实现
```java 方法一
public class Solution {
    boolean isBalanced = true;
    
    public boolean IsBalanced_Solution(TreeNode root) {
        if(root == null)
            return true;
        
        int left = getDepth(root.left);
        int right = getDepth(root.right);
        int diff = left - right;
        if(diff > 1 || diff < -1)
            return false;
        
        return IsBalanced_Solution(root.left) && IsBalanced_Solution(root.right);
    }
    
    private int getDepth(TreeNode root){
        if(root == null)
            return 0;
        
        int left = getDepth(root.left);
        int right = getDepth(root.right);
        
        return (left > right)?left+1:right+1;
    }
}
```

```java 方法三
public class Solution {
    boolean isBalanced = true;
    
    public boolean IsBalanced_Solution(TreeNode root) {
        getDepth(root);
        return isBalanced;
    }
    
    private int getDepth(TreeNode root){
        if(root == null)
            return 0;
        
        int left = getDepth(root.left);
        int right = getDepth(root.right);
        
        int diff = left - right;
        if(diff > 1 || diff < -1)
            isBalanced = false;
        
        return (left > right)?left+1:right+1;
    }
}
```