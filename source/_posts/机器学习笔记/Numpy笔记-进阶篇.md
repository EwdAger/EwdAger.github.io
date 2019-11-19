---
title: Numpy笔记-进阶篇
tags:
  - Python
  - Numpy
  - 数据分析
abbrlink: 87847f8
categories: 机器学习笔记
date: 2019-08-24 17:27:42
---

# 利用数组进行数据分析

`np.where`是三元表达式`x if condition else y`的矢量化版

```python
In [169]: arr
Out[169]:
array([[-2.09280349e-01, -2.08776777e+00,  1.18959772e+00,
        -1.30555812e-01],
       [-1.05658371e+00,  2.66633933e+00,  4.47784003e-01,
         5.22445402e-01],
       [ 9.67780972e-01, -1.00148828e+00, -1.83185363e-03,
         4.53542100e-01],
       [ 1.33135003e+00,  1.33233678e-01, -4.89156202e-01,
         1.16725743e+00]])

# 大于零替换成2，小于零替换成-2
In [170]: np.where(arr>0, 2, -2)
Out[170]:
array([[-2, -2,  2, -2],
       [-2,  2,  2,  2],
       [ 2, -2, -2,  2],
       [ 2,  2, -2,  2]])

# 大于零替换成2，小于零则不变
In [171]: np.where(arr>0, 2, arr)
Out[171]:
array([[-2.09280349e-01, -2.08776777e+00,  2.00000000e+00,
        -1.30555812e-01],
       [-1.05658371e+00,  2.00000000e+00,  2.00000000e+00,
         2.00000000e+00],
       [ 2.00000000e+00, -1.00148828e+00, -1.83185363e-03,
         2.00000000e+00],
       [ 2.00000000e+00,  2.00000000e+00, -4.89156202e-01,
         2.00000000e+00]])

```

## 数学和统计方法

以下方法可以在对某个轴向的数据进行统计，（axis=1,纵向；axis=0，横向）

```python
In [24]: arr
Out[24]:
array([[ 0,  1,  2,  3],
       [ 4,  5,  6,  7],
       [ 8,  9, 10, 11],
       [12, 13, 14, 15],
       [16, 17, 18, 19]])

In [25]: arr.sum()
Out[25]: 190

In [26]: arr.sum(axis=1)
Out[26]: array([ 6, 22, 38, 54, 70])

In [27]: arr.sum(axis=0)
Out[27]: array([40, 45, 50, 55])


```

方法 | 说明
--- | ---
sum | 对数组中所有或者某个轴向的数据进行求和，零长度的数组sum为0
mean | 算数平均值，零长度的数组mean为NaN
std、var | 标准差、方差
min、max | 最小值、最大值
argmin、argmax | 最小、最大值索引
cumsum | 所有元素的累计和
cumprod | 所有元素累计积

## 用于布尔型数组的方法

用于上面的方法中，布尔值会被强制转换成1和0。因此可以使用sum对布尔型数组的True值进行计数。

```python
In [38]: arr > 5
Out[38]:
array([[False, False, False, False],
       [False, False,  True,  True],
       [ True,  True,  True,  True],
       [ True,  True,  True,  True],
       [ True,  True,  True,  True]])

In [40]: (arr>5).sum()
Out[40]: 14
```

更有`any`和`all`方法可以判断布尔型数组中是否存在/全是`True`

```python
In [41]: bools = np.array([False, False, True, True, False])

In [42]: bools.any()
Out[42]: True

In [43]: bools.all()
Out[43]: False
```

## 排序

Numpy数组也可以使用`sort`方法就地排序，而`np.sort()`方法生成的是数组的副本，两者都可以在多维数组任意一个轴向上排序。

```python

In [79]: arr
Out[79]:
array([[20, 19, 18, 17],
       [16, 15, 14, 13],
       [12, 11, 10,  9],
       [ 8,  7,  6,  5],
       [ 4,  3,  2,  1]])

In [80]: arr.sort(axis=1)

In [81]: arr
Out[81]:
array([[17, 18, 19, 20],
       [13, 14, 15, 16],
       [ 9, 10, 11, 12],
       [ 5,  6,  7,  8],
       [ 1,  2,  3,  4]])

In [82]: arr.sort(axis=0)

In [83]: arr
Out[83]:
array([[ 1,  2,  3,  4],
       [ 5,  6,  7,  8],
       [ 9, 10, 11, 12],
       [13, 14, 15, 16],
       [17, 18, 19, 20]])

```

## 唯一化以及其他的集合逻辑

`np.unique()`方法可计算数组中的唯一值；`np.in1d()`可测试数组值得成员资格，返回布尔数组。

```python
# 唯一值
In [89]: names = np.array(['Bob', 'Bob', 'Bob', 'Will', 'Will', 'Joe'])

In [90]: np.unique(names)
Out[90]: array(['Bob', 'Joe', 'Will'], dtype='<U4')

# 成员资格
In [91]: values = np.array([1,2,3,3,3,4,5])

In [92]: np.in1d(values, [1,2,4])
Out[92]: array([ True,  True, False, False, False,  True, False])
```

方法 | 说明
--- | ---
unique(x) | 计算x中的唯一值，并返回有序结果
intersect1d(x, y) | 计算x, y中的公共元素，并返回有序结果
union1d(x, y) | 计算x, y的并集，返回有序结果
in1d(x, y) | 得到一个"x元素是否包含于y"的布尔型数组
setdiff1d(x, y) | 集合的差，即元素在x中且不在y中
setxor1d(x, y) | 集合的对称差，即存在于一个数组中但不同时存在于两个数组中的元素（异或）

# 线性代数

emmm。。反正看不懂，就先记个函数叭

函数 | 说明
--- | ---
diag | 以一维数组的形式返回方阵的对角线（或非对角线）元素，或将一维数组转换为方阵（非对角线元素为0）
dot | 矩阵乘法
trace | 计算对角线元素的和
det | 计算矩阵行列式
eig | 计算方阵的本征值和本征向量
inv | 计算方阵的逆
pinv | 计算矩阵的Moore-Penrose伪逆
qr | 计算QR分解
svd | 计算奇异值分解
solve | 解线性方程组Ax=b，其中A为一个方阵
lstsq | 计算Ax=b的最小二乘解

# 随机数生成

`numpy.random`效率比Python标准库的随机快的多

函数 | 说明
--- | ---
seed | 确定随机生成器的种子
permutation | 返回一个序列的随机排列或返回一个随机排列范围
shuffle | 对一个序列就地随机排列
rand | 产生均匀分布的样本值
randint | 从给定的上下限范围内随机选取整数
randn | 产生正态分布（平均值0，标准差1）的样本值
binomial | 产生二项分布的样本值
normal | 产生正态（高斯）分布的样本值
beta | 产生Beta分布的样本值
chisquare | 产生卡方分布的样本值
gamma | 产生Gamma分布的样本值
uniform | 产生[0, 1)中均匀分布的样本值

# 范例：随机漫步

**随机漫步理论(Random Walk Theory)**认为，证券价格的波动是随机的，像一个在广场上行走的人一样，价格的下一步将走向哪里，是没有规律的。证券市场中，价格的走向受到多方面因素的影响。一件不起眼的小事也可能对市场产生巨大的影响。从长时间的价格走势图上也可以看出，价格的上下起伏的机会差不多是均等的。

## Python标准库实现

```python
import random

position = 0
walk = [position]
steps = 1000
for i in range(steps):
  step = 1 if random.randint(0, 1) else -1
  position += step
  walk.append(position)

# 输出最大值
print(max(walk))

# 输出最小值
print(min(walk))

# 输出第一个到10时的索引
start, end = 0, 10
for k, v in enumerate(walk):
  if start == end:
    print(k)
    break
  start += v
  if k == len(walk):
    print("最终都没有到10")

```

## 使用Numpy

```python

In [98]: nsteps = 1000

# 随机生成nsteps个[0,2)之间的随机值
In [99]: draws = np.random.randint(0, 2, size=nsteps)

In [100]: steps = np.where(draws>0, 1, -1)

# 元素累计和的数组
In [101]: walk = steps.cumsum()

In [104]: walk.min()
Out[104]: -10

In [105]: walk.max()
Out[105]: 28

# 得到数组中第一个最大值的索引(第一个True)
In [106]: (walk >= 10).argmax()
Out[106]: 171

```

## 总结

很明显的可以看出，使用Numpy代码更加优雅易读，且通过IPython的`%timeit`测试两个版本的速度，使用标准库的平均时间为`4.57 ms`，而使用Numpy的平均时间为`2.68 ms`。