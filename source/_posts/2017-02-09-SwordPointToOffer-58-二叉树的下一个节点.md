---
title: 'SwordPointToOffer(58): 二叉树的下一个节点'
date: 2017-02-09 23:14:19
categories:
- SwordPointToOffer
---

# 题目
给定一个二叉树和其中的一个节点，请找出中序遍历顺序的下一个节点并且返回。注意，树中的节点不仅包含左右子节点，同时包含指向父节点的指针。

# 分析
如果一个节点有右子树，那么它的下一个节点就是它的右子树中的左子节点。也就是说右子节点出发一直沿着指向左子节点的指针，我们就能找到它的下一个节点。

接着我们分析一个节点没有右子树的情形。如果节点是它父节点的左子节点，那么它的下一个节点就是它的父节点。

如果一个节点既没有右子树，并且它还是它父节点的右子节点，这种情形就比较复杂。我们可以沿着指向父节点的指针一直向上遍历，直到找到一个是它父节点的左子节点的节点。如果这样的节点存在，那么这个节点的父节点就是我们要找的下一个节点。

# 实现
```java
public class NextNodeInBinaryTree {

    class TreeLinkNode {
        int val;
        TreeLinkNode left = null;
        TreeLinkNode right = null;
        TreeLinkNode parent = null;

        TreeLinkNode(int val) {
            this.val = val;
        }
    }
    
    public TreeLinkNode handle(TreeLinkNode node){
        if (node == null)
            return null;
        
        TreeLinkNode target = null;
        if (node.right != null){
            target = node.right;
            while (target.left != null){
                target = target.left;
            }
        }else if (node.parent != null){
            target = target.parent;
            TreeLinkNode cur = node;
            while (target != null && target.left != cur){
                cur = target;
                target = target.parent;
            }
        }
        return target;
    }
}
```