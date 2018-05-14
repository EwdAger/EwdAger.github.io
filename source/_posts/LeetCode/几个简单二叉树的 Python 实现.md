---
title: 几个简单二叉树的 Python 实现
tags:
  - LeetCode
  - Python
  - 二叉树
  - 数据结构
categories: LeetCode刷题总结
abbrlink: 3eeec910
date: 2018-05-14 20:14:00
---

## 二叉树的基本实现

```python
 class TreeNode:
     def __init__(self, x):
         self.val = x
         self.left = None
         self.right = None
```

<!--more-->
## [617.合并二叉树](https://leetcode-cn.com/problems/merge-two-binary-trees/description/)

![合并二叉树](https://gvoidy-1251878576.cos.ap-chengdu.myqcloud.com/leetcode-617.png)

```python
class Solution:
    def mergeTrees(self, t1, t2):
        """
        :type t1: TreeNode
        :type t2: TreeNode
        :rtype: TreeNode
        """
        if not t1:
            return t2
        if not t2:
            return t1
        t1.val += t2.val
        t1.left = self.mergeTrees(t1.left, t2.left)
        t1.right = self.mergeTrees(t1.right, t2.right)
        return t1     
```

## [104.二叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/description/)

![二叉树的最大深度](https://gvoidy-1251878576.cos.ap-chengdu.myqcloud.com/leetcode-104.png)

```python
class Solution:
    def maxDepth(self, root):
        """
        :type root: TreeNode
        :rtype: int
        """
        if root == None:
            return 0
        ldepth = Solution.maxDepth(self, root.left)
        rdepth = Solution.maxDepth(self, root.right)
        return max(ldepth, rdepth) + 1
```

## [108.将有序数组转换为二叉搜索树](https://leetcode-cn.com/problems/convert-sorted-array-to-binary-search-tree/description/)

![将有序数组转换为二叉搜索树](https://gvoidy-1251878576.cos.ap-chengdu.myqcloud.com/leetcode-108.png)

```python
class Solution:
    def sortedArrayToBST(self, nums):
        """
        :type nums: List[int]
        :rtype: TreeNode
        """
        if len(nums) == 0:
            return None
        if len(nums) == 1:
            return TreeNode(nums[0])
        if len(nums) == 2:
            tree = TreeNode(nums[1])
            tree.left = TreeNode(nums[0])
            return tree
        root = len(nums)//2
        tree = TreeNode(nums[root])
        tree.left = self.sortedArrayToBST(nums[0:root])
        tree.right = self.sortedArrayToBST(nums[root+1:])
        return tree
```