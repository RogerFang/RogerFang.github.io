---
title: 'SwordPointToOffer(60): 把二叉树打印成多行'
date: 2017-02-10 10:22:35
categories:
- SwordPointToOffer
---

# 题目
从上到下按层打印二叉树，同一层的节点按从左到右的顺序打印，每一层打印到一行。

# 分析
二叉树的层序遍历，需要借助队列完成。

本题需要注意的问题，将每层分行打印，需要两个变量：
* 一个变量表示在当前层中还没有打印的节点数；
* 一个变量表示下一层节点的数目。

# 实现
```java
public class PrintTreeInLines {
    
    class TreeNode {
        int val = 0;
        TreeNode left = null;
        TreeNode right = null;

        public TreeNode(int val) {
            this.val = val;

        }

    }

    public ArrayList<ArrayList<Integer>> handle(TreeNode root) {
        if(root == null)
            return null;
        
        ArrayList<ArrayList<Integer>> list = new ArrayList<>();
        LinkedList<TreeNode> queue = new LinkedList<>();
        queue.add(root);
        int current = 1;
        int next = 0;
        TreeNode node;
        ArrayList<Integer> levelList = new ArrayList<>();
        while (queue.size() > 0){
            // 从队首开始删
            node = queue.remove();
            levelList.add(node.val);
            if (node.left != null){
                queue.add(node.left);
                next++;
            }
            if (node.right != null){
                queue.add(node.right);
                next++;
            }
            current--;
            if (current == 0){
                list.add(levelList);
                levelList = new ArrayList<>();
                current = next;
                next = 0;
            }
        }
        return list;
    }
}
```