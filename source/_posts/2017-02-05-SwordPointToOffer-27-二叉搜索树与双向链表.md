---
title: 'SwordPointToOffer(27): 二叉搜索树与双向链表'
date: 2017-02-05 14:38:20
categories:
- SwordPointToOffer
---

# 题目
输入一棵二叉搜索树，将该二叉搜索树转换成一个排序的双向链表。要求不能创建任何新的节点，只能调整树中节点指针的指向。

# 分析
使用二叉树的中序遍历，并更改原节点的`left`和`right`引用指向。

> 注意：更改链表尾节点的引用，需要声明一个相应的全局变量。

# 实现
```java
static class TreeNode {
    int val = 0;
    TreeNode left = null;
    TreeNode right = null;

    public TreeNode(int val) {
        this.val = val;

    }

}

TreeNode tailNodeInList = null;

public TreeNode convert(TreeNode pRootOfTree) {
    if(pRootOfTree == null)
        return null;
    doConvert(pRootOfTree);

    TreeNode headNodeInList = tailNodeInList;
    while(headNodeInList != null && headNodeInList.left != null){
        headNodeInList = headNodeInList.left;
    }
    return headNodeInList;
}

private  void doConvert(TreeNode node){
    if(node == null)
        return;

    if(node.left != null){
        doConvert(node.left);
    }

    node.left = tailNodeInList;
    if(tailNodeInList != null){
        tailNodeInList.right = node;
    }
    tailNodeInList = node;

    if(node.right != null){
        doConvert(node.right);
    }
}
```