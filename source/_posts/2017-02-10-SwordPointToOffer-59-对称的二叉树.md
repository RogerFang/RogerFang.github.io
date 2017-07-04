---
title: 'SwordPointToOffer(59): 对称的二叉树'
date: 2017-02-10 10:07:22
categories:
- SwordPointToOffer
---

# 题目
请实现一个函数，用来判断一颗二叉树是不是对称的。注意，如果一个二叉树同此二叉树的镜像是同样的，定义其为对称的。

# 分析
通常我们有三种不同的二叉树遍历算法，即前序遍历、中序遍历和后序遍历。在这三种遍历算法中，都是先遍历左子结点再遍历右子结点。我们是否可以定义一种遍历算法，先遍历右子结点再遍历左子结点？比如我们针对前序遍历定义一种对称的遍历算法，即先遍历父节点，再遍历它的右子结点，最后遍历它的左子结点。

我们发现可以用过比较二叉树的**前序遍历序列**和**对称前序遍历序列**来判断二叉树是不是对称的。如果两个序列一样，那么二叉树就是对称的。

# 实现
```java
public class SymmetricalBinaryTree {

    class TreeNode {
        int val = 0;
        TreeNode left = null;
        TreeNode right = null;

        public TreeNode(int val) {
            this.val = val;

        }

    }

    public boolean isSymmetrical(TreeNode root) {
        if (root == null)
            return true;
        return isSymmetrical(root.left, root.right);
    }

    private boolean isSymmetrical(TreeNode left, TreeNode right) {
        if (left == null && right == null)
            return true;
        if (left == null || right == null)
            return false;
        if (left.val != right.val)
            return false;

        return isSymmetrical(left.left, right.right) && isSymmetrical(left.right, right.left);
    }
}
```