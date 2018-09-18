---
title: 慎用 Python list 乘法
tags:
  - Python
categories: python学习心得&备忘
abbrlink: 995b78a8
date: 2018-09-18 15:04:10
---

# 误用 list 乘法
今天刷 LeetCode 碰到一个水题[转置矩阵](https://leetcode-cn.com/problems/transpose-matrix/description/), 这不就是先生成个空的倒置矩阵再填结果嘛，没多想就用 list 乘法上手就写。

```python
class Solution:
    def transpose(self, A):
        """
        :type A: List[List[int]]
        :rtype: List[List[int]]
        """
        x, y = len(A), len(A[0])
        
        buff =[[0] * x] * y
        for i in range(x):
            for j in range(y):
                buff[j][i] = A[i][j]
        return buff
```

看似结果非常正确，但是样例输出了一个很奇怪的结果。

<!--more-->
```python
我的输入:
	[[1,2,3],[4,5,6]]

我的答案:
	[[3,6],[3,6],[3,6]]

标准答案：
	[[1,4],[2,5],[3,6]]
```

赶紧在第12行前加上`print(buff)`一看

```python
我的输入:
	[[1,2,3],[4,5,6]]

标准输出：
	[[0, 0], [0, 0], [0, 0]]
	[[1, 0], [1, 0], [1, 0]]
	[[2, 0], [2, 0], [2, 0]]
	[[3, 0], [3, 0], [3, 0]]
	[[3, 4], [3, 4], [3, 4]]
	[[3, 5], [3, 5], [3, 5]]
```
# 我的猜测与验证

发现果然列表里的每一个子元素都相等了，猜测可能是 **只复制了值的引用，而不是新建了一个对象**,接下来就是验证。

```python
>>> a = [[0]*6]*4
>>> a
[[0, 0, 0, 0, 0, 0], [0, 0, 0, 0, 0, 0], [0, 0, 0, 0, 0, 0], [0, 0, 0, 0, 0, 0]]
>>> id(a[0])
1423765355080
>>> id(a[1])
1423765355080
```

发现不出所料果然是这样。

# 正确姿势

list 的乘法是能很方便的构建一个全为重复元素的**一维**列表方法，但在多维情况下非常容易出错。 所以说慎用 list 乘法！！！想构建 list 老老实实给我用列表生成器去。不要嫌写的太多不 `pythonic`

```python
class Solution:
    def transpose(self, A):
        """
        :type A: List[List[int]]
        :rtype: List[List[int]]
        """
        x, y = len(A), len(A[0])
        
        buff =[[0 for i in range(x)] for i in range(y)]
        for i in range(x):
            for j in range(y):
                buff[j][i] = A[i][j]
        return buff
```
