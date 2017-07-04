---
title: 'SwordPointToOffer(25): 二叉树中和为某一值的路径'
date: 2017-02-05 12:21:16
categories:
- SwordPointToOffer
---

# 题目
输入一棵二叉树和一个整数，打印出二叉树中节点值的和为输入整数的所有路径。从树的根节点开始往下一直到叶节点所经过的节点形成一条路径。

# 分析
由于路径是从根结点出发到叶结点， 也就是说路径总是以根结点为起始点，因此我们首先需要遍历根结点。在树的前序、中序、后序三种遍历方式中，只有前序遍历是首先访问根结点的。

当用前序遍历的方式访问到某一结点时， 我们把该结点添加到路径上，并累加该结点的值。如果该结点为叶结点并且路径中结点值的和刚好等于输入的整数， 则当前的路径符合要求，我们把它打印出来。如果当前结点不是叶结点，则继续访问它的子结点。当前结点访问结束后，递归函数将自动回到它的父结点。因此**在函数退出之前要在路径上删除当前结点并减去当前结点的值**，以确保返回父结点时路径刚好是从根结点到父结点的路径。

保存单条路径的数据结构是**栈**。

# 实现
```java
import java.util.ArrayList;
import java.util.Stack;

public class PathInTree {
    
    class TreeNode {
        int val = 0;
        TreeNode left = null;
        TreeNode right = null;

        public TreeNode(int val) {
            this.val = val;

        }

    }

    public ArrayList<ArrayList<Integer>> FindPath(TreeNode root, int target) {
        ArrayList<ArrayList<Integer>> paths = new ArrayList<>();
        if(root != null){
            Stack<Integer> stack = new Stack<>();
            doFindPath(root, paths, stack, 0, target);
        }
        return paths;
    }

    private void doFindPath(TreeNode node, ArrayList<ArrayList<Integer>> paths, Stack<Integer> path, int curSum, int expectSum){
        if(node == null){
            return;
        }

        curSum += node.val;
        path.push(node.val);
        if(node.left != null){
            doFindPath(node.left, paths, path, curSum, expectSum);
        }
        if(node.right != null){
            doFindPath(node.right, paths, path, curSum, expectSum);
        }
        if(node.left == null && node.right == null){
            if(curSum == expectSum){
                ArrayList<Integer> list = new ArrayList<>();
                for(int i: path){
                    list.add(i);
                }
                paths.add(list);
            }
        }

        path.pop();
    }
}
```