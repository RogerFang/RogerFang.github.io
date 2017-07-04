---
title: 'SwordPointToOffer(18): 树的子结构'
date: 2017-01-29 15:22:42
categories:
- SwordPointToOffer
---

# 题目
输入两棵二叉树A和B，判断B是不是A的子结构。

# 分析
整个判断过程分为两步：
1. 在树A中查找与根节点的值一样的节点R，实际上是二叉树的遍历。
2. 判断树A中以R为根节点的子树是不是和树B具有相同的结构。

# 实现
```java
public class HasSubTree {
    class TreeNode {
        int val = 0;
        TreeNode left = null;
        TreeNode right = null;

        public TreeNode(int val) {
            this.val = val;

        }
    }

    public boolean has(TreeNode root1, TreeNode root2) {
        boolean result = false;
        if(root1 != null && root2 != null){
            if(root1.val == root2.val){
                result = doHasSubTree(root1, root2);
            }
            if(!result){
                result = has(root1.left, root2);
            }
            if(!result){
                result = has(root1.right, root2);
            }
        }
        return result;
    }

    private boolean doHasSubTree(TreeNode root1, TreeNode root2){
        if(root2 == null){
            return true;
        }

        if(root1 == null){
            return false;
        }

        if(root1.val != root2.val){
            return false;
        }

        return doHasSubTree(root1.left, root2.left) && doHasSubTree(root1.right, root2.right);
    }
}
```