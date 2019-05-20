---
layout: post
title: Tree结构中的global variable 加 recursion 解法
date: 2016-03-22
tag: Leetcode
---

### 左右子树问题

有一类问题,从左右子树传回的值不一定是最终的结果,而是某种状态.这样的题,我们可以用设立global variable加上 recursion的方法来做.


##### 968 Binary Tree Camera
> 大意: 一个节点上camera可以监视父节点,本节点,及子节点,问一个tree中最少需要多少个camera可以监视所有node?

```python


```

### Traversal 问题

Traversal 问题主要有三种: pre_order, in_order, 和 post_order.当然还有小部分题型是关于level traversal及其它traversal order的.我们主要看这三大种.对于这几种traversal,我觉得最重要要记住的是,采用recursion的思想,建立一个global variable, 随着节点的traversal变化而被改变,同时父节点不要管子节点是如何实现的,只要管好自己就好了.我们来看几题.

##### 230 Kth Smallest Element in a BST
> 大意: 找出一个BST中第k小的数字.

解答: 我们可以用in_order traversal 把所有的顺序找出来,但是这种效率不太高,尤其是如果需要更新node的时候.我们采用DFS的思想来解答这题.

```python
class Solution(object):
    def kthSmallest(self, root, k):
        status = [-1, k]          # 建立一个reference,然后节点都可以更改它.第一个数字是记录结果,第二个数字记录进行到第几个了
        self.in_order(root, status)      
        return status[0]

    def in_order(self, root, status):
        if not root:              # 第一步: handle 虚节点
            return status

        self.in_order(root.left, status)       # 第二步, 左子树

        if status[1] == 0:
            return

        if status[1] == 1:                     # 第三步: 自己
            status[1] -= 1
            status[0] = root.val
            return

        status[1] -= 1
        self.in_order(root.right, status)      # 第四步: 右子树.
        # 完结
```
