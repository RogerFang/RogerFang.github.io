---
title: 'SwordPointToOffer(23): 从上往下打印二叉树'
date: 2017-02-05 10:19:10
categories:
- SwordPointToOffer
---

# 题目
从上往下打印出二叉树的每个节点，同一层的节点按照从左往右的顺序打印。如下图中的二叉树，则依次打印出8、6、10、5、7、9、11。
![](/images/swordpointtooffer/t23.png)

# 分析
通过队列来保存遍历的节点。

# 实现
```java
public static ArrayList<Integer> print(TreeNode node){
    if (node == null)
        return null;

    ArrayList<Integer> list = new ArrayList<>();
    Queue<TreeNode> queue = new LinkedList<>();
    queue.add(node);
    TreeNode curNode;
    while (!queue.isEmpty()){
        curNode = queue.remove();
        list.add(curNode.val);

        if (curNode.left != null){
            queue.add(curNode.left);
        }
        if (curNode.right != null){
            queue.add(curNode.right);
        }
    }
    return list;
}
```