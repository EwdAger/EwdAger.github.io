---
title: Python 查漏补缺之字符串
tags:
  - LeetCode
categories: LeetCode刷题总结
abbrlink: 4be7f132
date: 2018-04-27 14:33:00
---

## count 方法

```python
str.count(sub, start= 0,end=len(string))
```

**参数**

- sub -- 搜索的子字符串
- start -- 字符串开始搜索的位置。默认为第一个字符,第一个字符索引值为0。
- end -- 字符串中结束搜索的位置。字符中第一个字符的索引为 0。默认为字符串的最后一个位置。

**返回值**

该方法返回子字符串在字符串中出现的次数。

<!--more-->

### LeetCode 相关题目 [461. 汉明距离](https://leetcode-cn.com/problems/hamming-distance/description/)

**最优解法**

```python
class Solution:
    def hammingDistance(self, x, y):

        return(bin(x^y).count('1'))
```