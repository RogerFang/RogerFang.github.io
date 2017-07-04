---
title: 'SwordPointToOffer(62): 序列化二叉树'
date: 2017-02-10 11:35:56
categories:
- SwordPointToOffer
---

# 题目
请实现两个函数，分别用来序列化和反序列化二叉树。

# 分析
## 方法一
我们知道可以从前序遍历和中序遍历构造出一棵二叉树。受此启发，我们可以先把一棵二叉树序列化成一个前序遍历序列和一个中序序列，然后再反序列化时通过这两个序列重构出原二叉树。

这个思路有两个**缺点**。一个缺点是该方法要求二叉树中不能用有数值重复的节点。另外只有当两个序列中所有数据都读出后才能开始反序列化。如果两个遍历序列的数据是从一个流里读出来的，那就可能需要等较长的时间。

## 方法二
实际上如果二叉树的序列化是从根节点开始的话，那么相应的反序列化在根节点的数值读出来的时候就可以开始了。因此我们可以根据前序遍历的顺序来序列化二叉树，因为前序遍历是从根结点开始的。当在遍历二叉树碰到 NULL 指针时，这些 NULL 指针序列化成一个特殊的字符（比如‘$’）。另外，节点的数值之间要用一个特殊字符（比如’,’）隔开。

# 实现
```java
public class SerializeBinaryTree {

    class TreeNode {
        int val = 0;
        TreeNode left = null;
        TreeNode right = null;

        public TreeNode(int val) {
            this.val = val;
        }
    }
    
    private static final String NULL_NODE = "$";
    private static final String SEPARATOR = ",";
    
    public String serialize(TreeNode root){
        if (root == null)
            return "";
        
        StringBuilder sb = new StringBuilder();
        doSerialize(root, sb);
        return sb.toString();
    }
    
    private void doSerialize(TreeNode node, StringBuilder sb){
        if (node == null){
            sb.append(NULL_NODE);
            sb.append(SEPARATOR);
        }else {
            sb.append(node.val);
            sb.append(SEPARATOR);
            doSerialize(node.left, sb);
            doSerialize(node.right, sb);
        }
    }
    
    public TreeNode deserialize(String str){
        if (str == null || str.length() <= 0)
            return null;
        String[] splits = str.split(SEPARATOR);
        return doDeserialize(splits);
    }
    
    private int index = -1;
    
    private TreeNode doDeserialize(String[] strs){
        index++;
        if (!strs[index].equals(NULL_NODE) && !strs[index].equals("") && index < strs.length){
            TreeNode root = new TreeNode(Integer.parseInt(strs[index]));
            root.left = doDeserialize(strs);
            root.right = doDeserialize(strs);
            return root;
        }
        return null;
    }
}
```