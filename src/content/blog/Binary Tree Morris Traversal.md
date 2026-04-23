---
title: 二叉树的 Morris 遍历
description: 'Morris 遍历实现了在线性时间内，使用常数空间完成二叉树的前序、中序、后序三种遍历。'
publishDate: 2026-4-14
tags:
  - algorithm
  - morris
---

## 前言

Morris 遍历使用巧妙的方法实现了在线性时间内，使用常数空间完成二叉树的前序、中序、后序三种遍历。
由 J. H. Morris 在 1979 年的论文「Traversing Binary Trees Simply and Cheaply」中首次提出，因此被称为 Morris 遍历。

## 原理

在常数空间下完成三种遍历，意味着不可以用简单的递归，只能使用迭代。试着思考一下，迭代要如何实现。
聪明的你很快会发现迭代很容易就可以沿着树的某条枝向下寻找，但是你发现无法回头去遍历其他分支。
理所当然你会想到我需要维护一个变量 prev 使得我可以“回头”。很快又有一个问题，一个变量我只能“回头”一次，树的高度大于 1 时，便不能实现。
因为 prev 一直会被覆盖只能记录上一个值。

此时我们需要思考，我们什么时候需要“回头”，常规我们都是从左遍历到右，**先从当前节点遍历左子树，然后返回根节点，最后遍历右子树**，也就是中序遍历。

那么我们遍历一颗树，不在意其子树，只看它本身，对它而言的第一次也是唯一一次“回头”就是左子树遍历完成回到根节点。
没看懂没关系，看下面的例子：

我们有树 `root = [1, 2, 3]`， 首先我们从当前节点访问左子树（2），完成后返回根节点（1）访问遍历右子树（3）。
那么我们的唯一一次“回头”也就是左子树（2）到根节点（1）。

![例子](https://cos.arshe.cn/Binary%20Tree%20Morris%20Traversal/%E4%BE%8B%E5%AD%90.png)

我们回到前面的思考，我们刚开始想维护一个变量 prev 来实现回头，回头想法没有错，但实现的方式错了。
树的节点本就拥有指针，我们将左子树的最后一个节点（肯定是叶子节点）的右孩子的指针指向根节点，那么当我们遍历完成左子树后，可以直接根据最后一个节点的右孩子指针回到根节点。

我们需要预处理这个右孩子指针，左子树的根节点一直向右子树遍历，找到的叶子节点就是左子树的最后一个节点。
因为我们的遍历方式是先左子树，然后根节点，最后右子树。

综上所述，我们通过 Morris 遍历的方法遍历一个二叉树的完整流程如下：

判断当前节点 `node` 是否有左子树：

1. 有左子树，声明一个 `predecessor` 指针赋值为 `node.left`。`predecessor` 一直向右子树遍历到底即可找到 `node` 左子树的最后一个节点。然后进行如下操作：
    - 若 `predecessor` 的右子树为空，将 `predecessor` 的右子树指向 `node`。`node` 遍历其左子树即 `node = node.left`。
    - 若 `predecessor` 的右子树指向 `node`，则说明 `node` 的左子树已经遍历完成，`node` 遍历其右子树即 `node = node.right` 并将 `predecessor` 的右子树置为 `null`。
2. 没有左子树，则向右遍历即 `node = node.right`。
3. 重述上诉操作，直至访问完整棵树。

代码实现：

```java
while(node != null){
    if(node.left != null){
        TreeNode predecessor = node.left;
        while(predecessor.right != null && predecessor.right != node){
            predecessor = predecessor.right;
        }
        if(predecessor.right == null){
            predecessor.right = node;
            node = node.left;
        }else{
            predecessor.right = null;
            node = node.right;
        }
    }else{
        node = node.right;
    }
}
```

那么前序、中序、后序三种遍历只是值输出的时机不同。

## 前序遍历

```java
while(node != null){
    if(node.left != null){
        TreeNode predecessor = node.left;
        while(predecessor.right != null && predecessor.right != node){
            predecessor = predecessor.right;
        }
        if(predecessor.right == null){
            System.out.println(node.val);    // [!code ++]
            predecessor.right = node;
            node = node.left;
        }else{
            predecessor.right = null;
            node = node.right;
        }
    }else{
        System.out.println(node.val);   // [!code ++]
        node = node.right;
    }
}
```

## 中序遍历

```java
while(node != null){
    if(node.left != null){
        TreeNode predecessor = node.left;
        while(predecessor.right != null && predecessor.right != node){
            predecessor = predecessor.right;
        }
        if(predecessor.right == null){
            predecessor.right = node;
            node = node.left;
        }else{
            System.out.println(node.val);    // [!code ++]
            predecessor.right = null;
            node = node.right;
        }
    }else{
        System.out.println(node.val);   // [!code ++]
        node = node.right;
    }
}
```

## 后序遍历

后序遍历是最特殊的，需要记录值的时机是在当前节点的左子树遍历完成时，记录左子树一直沿着其右子树遍历的值。

```java
public List<Integer> postorderTraversal(TreeNode root) {
    List<Integer> res = new ArrayList<>();
    TreeNode tempRoot = root;
    while(root != null){
        if(root.left != null){
            TreeNode prodcessor = root.left;
            while(prodcessor.right != null && prodcessor.right != root){
                prodcessor = prodcessor.right;
            }
            if(prodcessor.right == null){
                prodcessor.right = root;
                root = root.left;
            }else{
                prodcessor.right = null;
                TreeNode temp = root.left;
                recursivelyAdd(res, root.left); // [!code ++]
                root = root.right;
            }
        }else{
            root = root.right;
        }
    }
    recursivelyAdd(res, tempRoot); // [!code ++]
    return res;
}

public void recursivelyAdd(List<Integer> res, TreeNode node){ // [!code ++]
    if(node == null){ // [!code ++]
        return; // [!code ++]
    } // [!code ++]
    recursivelyAdd(res, node.right);  // [!code ++]
    res.add(node.val);  // [!code ++]
}
```