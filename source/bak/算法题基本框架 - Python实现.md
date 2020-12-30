---
title: 算法题基本框架 - Python实现
tags:
  - LeetCode
  - Python
  - 二叉树
  - 数据结构
categories: LeetCode刷题总结
abbrlink: 5cb682e4
date: 2020-12-08 14:51:00
---

# 二叉树

## BFS

```python
# 

from collections import deque

def BFS(root: Node, target: Node):
	queue = deque()
	# 记录访问的节点避免走回头路
	vistied = []

	# 起点在循环外入队
	queue.append(root)
	vistied.append(root)
	step = 0

	while queue:
		# 先记录此时队列长度，避免将新入队元素算入本次循环
		length = len(queue)
		for i in range(length):
			cur = queue.popleft()

			# 判断是否到达终点（此处需改成符合题意的条件）
			if cur is target:
				return step

			# 将cur相邻节点入队，adj()泛指cur相邻节点
			for x in cur.adj():
				if x not in vistied:
					queue.append(x)
					vistied.append(x)
		step += 1
```
