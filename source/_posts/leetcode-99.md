---
title: 二叉树遍历-Morris中序遍历
date: 2020-08-08 17:42:36
tags: ["leetcode","二叉树","算法"]
categories: "学习"
mathjax: true
---
## Morris中序遍历

### 中序遍历

二叉树的中序(inorder)遍历，也就是二叉树最平常的遍历算法。对于树中每个节点，递归遍历左子树，之后遍历本节点，最后遍历右子树。其中分为迭代与递归两种等价写法，迭代实现需要手动维护一个节点栈。

这个遍历算法的时间复杂度为$O(N)$，其中N为二叉树的节点数，空间复杂度为$O(N)$(若用数组存储遍历序列),或$O(H)$,$H$为二叉树高度（用迭代实现，栈中存储结点）。



然而，存在一个**常数空间**算法来解决这个问题：

### Morris 遍历

<!-- more -->

Morris遍历算法只需要维护一个节点$pred$，它代表最终遍历序列中的前序节点。

算法的整体步骤如下：（假设当前遍历到的节点为$x$）

- 若$x$无左孩子，则可以将$x$输出，即$pred = x$,之后遍历$x$的右孩子。
- 若$x$有左孩子，则先寻找x左子树上的最右节点（也就是$x$在中序遍历中的前序节点），记为$predecessor$。其中：
  - 若$predecessor$的右孩子为空，则将它的右孩子指向$x$,之后遍历$x$的左孩子。
  - 若$predecessor$的右孩子不为空，它只能为$x$，则说明此时是第二次遍历节点$x$（即之前进行过上一步）, 因此将$predecessor$的右孩子置为空，输出$x$即$pred=x$，之后遍历$x$的右孩子
- 重复上述操作，直至访问完整棵树



**算法理解：**

改变$predecessor$的右孩子，使其指向$x$，便于遍历左子树后能遍历回节点$x$。每次条件判断后只遍历一个子节点，相当于将树形结构变成了线性结构，通过遍历每个节点2次来降低算法的空间复杂度。

时间复杂度：$O(2N) = O(N)$

空间复杂度：$O(1)$



### leetcode 99 恢复二叉搜索树

二叉搜索树中的两个节点被错误地交换。

请在不改变其结构的情况下，恢复这棵树。

示例 1:

输入: [1,3,null,null,2]

   1
  /
 3
  \
   2

输出: [3,1,null,null,2]

   3
  /
 1
  \
   2

示例 2:

输入: [3,1,4,null,null,2]

  3
 / \
1   4
   /
  2

输出: [2,1,4,null,null,3]

  2
 / \
1   4
   /
  3

进阶:

- 使用 O(n) 空间复杂度的解法很容易实现。
- 你能想出一个只使用常数空间的解决方案吗？

写的时候有点着急，代码有点乱

``` python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def recoverTree(self, root: TreeNode) -> None:
        """
        Do not return anything, modify root in-place instead.
        """
        wrongNode=[]
        pred = None
        def morrisInorder(root):
            nonlocal pred
            if root.left:
                predecessor = root.left
                while predecessor.right and predecessor.right!=root: predecessor = predecessor.right
                if not predecessor.right:
                    predecessor.right = root
                    morrisInorder(root.left)
                else:
                    if pred and pred.val > root.val:
                        wrongNode.append([pred,root])
                    predecessor.right = None
                    pred=root
                    if root.right:
                        morrisInorder(root.right)
            else:
                if pred and pred.val > root.val:
                    wrongNode.append([pred,root])
                pred=root
                if root.right:
                    morrisInorder(root.right)    
        morrisInorder(root)
        if len(wrongNode)==1:
            temp = wrongNode[0][1].val
            wrongNode[0][1].val = wrongNode[0][0].val
            wrongNode[0][0].val = temp
        if len(wrongNode)==2:
            temp = wrongNode[0][0].val
            wrongNode[0][0].val = wrongNode[1][1].val
            wrongNode[1][1].val = temp

```

