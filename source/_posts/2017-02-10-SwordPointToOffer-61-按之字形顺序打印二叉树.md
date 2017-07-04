---
title: 'SwordPointToOffer(61): 按之字形顺序打印二叉树'
date: 2017-02-10 11:03:36
categories:
- SwordPointToOffer
---

# 题目
请实现一个函数按照之字形打印二叉树，即第一行按照从左到右的顺序打印，第二层按照从右至左的顺序打印，第三行按照从左到右的顺序打印，其他行以此类推。

# 分析
按之字形顺序打印二叉树需要**两个栈**。我们在打印某一行结点时，把下一层的子结点保存到相应的栈里。如果当前打印的是奇数层，则先保存左子结点再保存右子结点到一个栈里；如果当前打印的是偶数层，则先保存右子结点再保存左子结点到第二个栈里。

# 实现
```java
public class PrintTreeinZigzag {

    class TreeNode {
        int val = 0;
        TreeNode left = null;
        TreeNode right = null;

        public TreeNode(int val) {
            this.val = val;

        }
    }

    public ArrayList<ArrayList<Integer>> handle(TreeNode root){
        if (root == null)
            return null;

        ArrayList<ArrayList<Integer>> list = new ArrayList<>();
        ArrayList<LinkedList<TreeNode>> stacks = new ArrayList<>();
        stacks.add(new LinkedList<TreeNode>());
        stacks.add(new LinkedList<TreeNode>());
        // 当前使用栈0/1
        int current = 0;

        stacks.get(current).add(root);
        ArrayList<Integer> levelList = new ArrayList<>();
        while (!stacks.get(current).isEmpty()){
            // 出栈
            TreeNode node = stacks.get(current).removeLast();
            levelList.add(node.val);
            if (current == 0){
                // right to left
                if (node.left != null)
                    stacks.get(1-current).add(node.left);
                if (node.right != null)
                    stacks.get(1-current).add(node.right);
            }else {
                // left to right
                if (node.right != null)
                    stacks.get(1-current).add(node.right);
                if (node.left != null)
                    stacks.get(1-current).add(node.left);
            }

            if (stacks.get(current).isEmpty()){
                current = 1- current;
                list.add(levelList);
                levelList = new ArrayList<>();
            }
        }
        return list;
    }
}
```