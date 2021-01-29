---
title: 算法题基本框架 - Python实现
tags:
  - LeetCode
  - Python
  - 二叉树
  - 数据结构
categories: LeetCode刷题总结
abbrlink: 5cb682e4
date: 2020-08-28 15:31:00
---

# 二叉树

## DFS

> 二叉搜索树的中序遍历即为这棵树叶子节点值从小到大的排列

```python
def dfs(root: TreeNode):
	if not root:
		return

	# 先序遍历
	dfs(root.left)
	# 中序遍历
	dfs(root.right)
	# 后续遍历
```

## BFS

```python
from collections import deque

def bfs(root: TreeNode, target: TreeNode):
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

# 回溯算法

例题: [46.全排列](https://leetcode-cn.com/problems/permutations/)

本质上为N叉树的遍历 + 决策
```python
class Solution:
    def __init__(self):
        self.ans = []
        self.sub_ans = []

    # 主函数
    def permute(self, nums: List[int]) -> List[List[int]]:
        self.helper(nums)
        return self.ans

    def helper(self, nums):
    	# 触发结束条件
        if len(self.sub_ans) == len(nums):
            self.ans.append(self.sub_ans)
            return

        for num in nums:
        	# 排除非法的选择
            if num in self.sub_ans:
                continue
            # 做选择
            self.sub_ans.append(num)
            # 进入决策树
            self.helper(nums)
            # 取消当前选择
            self.sub_ans = self.sub_ans[:-1]
```

# 二分查找

## 精确搜索

```python
def binary_search(nums, target):
    """
    :param nums: 递增的排序数组
    :param target: 目标
    """

    left, right = 0, len(nums) - 1
    while left <= right:
        mid = left + (right - left) / 2
        if nums[mid] < target:
            left = mid + 1
        elif nums[mid] > target:
            right = mid - 1
        elif nums[mid] == target:
            return mid

    return -1
```

## 查找左边界

```python
def left_search(nums, target):
    """
    :param nums: 递增的排序数组
    :param target: 目标
    """

    left, right = 0, len(nums) - 1
    while left <= right:
        mid = left + (right - left) / 2
        if nums[mid] < target:
            left = mid + 1
        elif nums[mid] > target:
            right = mid - 1
        elif nums[mid] == target:
            right = mid - 1

    if left >= len(nums) or nums[left] != target:
        return -1
    return left
```

## 查找右边界

```python
def right_search(nums, target):
    """
    :param nums: 递增的排序数组
    :param target: 目标
    """

    left, right = 0, len(nums) - 1
    while left <= right:
        mid = left + (right - left) / 2
        if nums[mid] < target:
            left = mid + 1
        elif nums[mid] > target:
            right = mid - 1
        elif nums[mid] == target:
            left = mid + 1

    if left < 0 or nums[left] != target:
        return -1
    return left
```

# 滑动窗口

## 有固定长度的目标

例题: [76.最小覆盖子串](https://leetcode-cn.com/problems/minimum-window-substring/)
```python
from collections import Counter


class Solution:
    def minWindow(self, s: str, t: str) -> str:
        """
        :param s: "ADOBECODEBANC"
        :param t: "ABC"
        """
        window = {}
        need = Counter(t)

        left, right = 0, 0
        is_valid = 0

        start = 0
        length = float("INF")

        while right < len(s):
            c = s[right]
            right += 1

            if c in need:
                window[c] = window.get(c, 0) + 1
                if window[c] == need[c]:
                    is_valid += 1

            while is_valid == len(need):
                if right - left < length:
                    start = left
                    length = right - left

                d = s[left]
                left += 1

                if d in need:
                    if window[d] == need[d]:
                        is_valid -= 1
                    window[d] = window[d] - 1

        return s[start: length+start] if length != float("INF") else ""
```

## 无固定长度的目标

例题: [3.无重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)
```python
class Solution:
    def lengthOfLongestSubstring(self, s: str) -> int:
        window = {}

        left, right = 0, 0
        max_window = 0

        while right < len(s):

            c = s[right]
            right += 1

            window[c] = window.get(c, 0) + 1

            while window[c] > 1:

                d = s[left]
                left += 1

                if d in window:
                    window[d] -= 1

            max_window = max(max_window, right - left)

        return max_window
```

# 并查集

通常情况下使用数组维护的并查集更省空间，因为直接定义了一个n条边的数组，使用下标来维护对应关系。但是遇到二维坐标时，用哈希维护的并查集更合适，因为可以把y映射到x取值范围外，使二维转化为一维。比如: [1584.连接所有点的最小费用](https://leetcode-cn.com/problems/min-cost-to-connect-all-points/)

## 哈希维护(邻接表)

```python
class UnionFind:
    def __init__(self):
        # 哈希维护的并查集在find()时添加边
        self.fa = {}
        # size 维护树的高度，尽量将小树连接到大树上
        self.size = {}
        # 连通分量(当前图有几个连通图),find()时+1，union()时-1
        self.count = 0

    def find(self, n):

        # 当前点不存在
        if n not in self.fa:
            self.fa[n] = n
            self.size[n] = 1
            self.count += 1

        while x != self.fa[x]:
            x = self.fa[x]
            self.fa[x] = self.fa[self.fa[x]]

        return x

    def union(self, p, q):
        root_p = self.find(p)
        root_q = self.find(q)

        if root_p == root_q:
            # 如果在并查集构建完成中要求查询是否有连通关系，可以直接if uf.union(p, q)判断
            return False

        if self.size[root_p] < self.size[root_q]:
            self.fa[root_p] = root_q
            self.size[root_q] += self.size[root_p]
        else:
            self.fa[root_q] = root_p
            self.size[root_p] += self.size[root_q]

        self.count -= 1

        return True

    def is_connect(self, p, q):
        # 如果在并查集构建完成后查询是否有连通关系则使用该函数
        root_p = self.find(p)
        root_q = self.find(q)

        return root_p == root_q
```

## 数组维护(使用下标维护的邻接矩阵，只有一维)

当需要连接的点为二维坐标时，可以用 `i * 列数 + j` 将二维投射到一维上来初始化

```python
class UnionFind:
    def __init__(self, n):
        # 因为初始化的时候需要确定点的数量，同时也可把连通分量设为点的数量，即每个点都不与其他点相连
        self.count = n
        self.fa = [n for n in range(n)]
        self.size = [1 for _ in range(n)]

    def find(self, x):

        while x != self.fa[x]:
            x = self.fa[x]
            self.fa[x] = self.fa[self.fa[x]]

        return x

    def union(self, p, q):
        root_p = self.find(p)
        root_q = self.find(q)

        if root_p == root_q:
            # 如果在并查集构建完成中要求查询是否有连通关系，可以直接if uf.union(p, q)判断
            return False

        if self.size[root_p] < self.size[root_q]:
            self.fa[root_p] = root_q
            self.size[root_q] += self.size[root_p]
        else:
            self.fa[root_q] = root_p
            self.size[root_p] += self.size[root_q]

        self.count -= 1

        return True

    def is_connect(self, p, q):
        # 如果在并查集构建完成后查询是否有连通关系则使用该函数
        root_p = self.find(p)
        root_q = self.find(q)

        return root_p == root_q
```

## 变体 Kruskal 算法(贪心+并查集)

求图的最小生成树

> 其算法流程为：
1. 将图 G=\{V,E\}G={V,E} 中的所有边按照长度由小到大进行排序，等长的边可以按任意顺序。
2. 初始化图 G'G′为 \{V,\varnothing\}{V,∅}，从前向后扫描排序后的边，如果扫描到的边 ee 在 G'G′中连接了两个相异的连通块,则将它插入 G'G′中。
3. 最后得到的图 G'G′就是图 GG 的最小生成树。

例题: [1584.连接所有点的最小费用](https://leetcode-cn.com/problems/min-cost-to-connect-all-points/)

```python
from typing import List


class UnionFind:
    def __init__(self, n):
        self.parent = [i for i in range(n)]

    def find(self, x):

        while x != self.parent[x]:
            x = self.parent[x]
            self.parent[x] = self.parent[self.parent[x]]

        return x

    def union(self, p, q):
        root_p = self.find(p)
        root_q = self.find(q)

        if root_p == root_q:
            return False
        self.parent[root_p] = root_q
        return True


class Solution:
    def minCostConnectPoints(self, points: List[List[int]]) -> int:
        edges = []

        for i in range(len(points)):
            for j in range(i+1, len(points)):
                edges.append((self.route_cost(points[i], points[j]), i, j))

        edges.sort()
        min_cost, num = 0, 0
        uf = UnionFind(len(points))

        for cost, x, y in edges:
            if uf.union(x, y):
                min_cost += cost
                num += 1
                
                if num == len(points):
                    break
        return min_cost
            

    @staticmethod
    def route_cost(a: List, b: List):
        return abs(abs(a[1] - b[1]) + abs(a[0] - b[0]))


if __name__ == "__main__":
    obj = Solution()
    ans = obj.minCostConnectPoints([[0, 0], [2, 2], [3, 10], [5, 2], [7, 0]])
    print(ans)
```

# 动态规划

几种dp数组的定义方法

- dp[i] 表示 以 nums[i] 这个坐标中的数结尾的最长递增子序列长度 [300.最长递增子序列](https://leetcode-cn.com/problems/longest-increasing-subsequence/)