---
title: 机器学习速查笔记-Numpy篇
tags:
  - Python
  - Numpy
  - 数据分析
  - 机器学习
abbrlink: ec7adaa1
categories: 机器学习笔记
date: 2019-09-05 16:11:42
---

# numpy

## np.unique(A)
对于一维数组或者列表，unique函数去除其中重复的元素，并按元素由大到小返回一个新的无元素重复的元组或者列表
```python
A = [1,1,2,3,4,4,5,5,6]
a = np.unique(A)
print(a)   # [1 2 3 4 5 6]
```

## np.random.rand（x…）
生成随机的指定维度的列表
```python
a=np.random.rand(4)
for var in range(10):
    print(a)
    a = np.random.rand(4)
```
结果：
```
[0.21849839 0.06253395 0.10794774 0.88207845]
[0.38262448 0.93590044 0.01358229 0.67018295]
[0.52202392 0.55000451 0.8633613  0.64067578]
[0.3782018  0.74330012 0.33034715 0.74607596]
[0.07583301 0.71505934 0.66028763 0.54293845]
```
<!--more-->
## np.random.uniform(low,high,size)
[参考原文](https://blog.csdn.net/u013920434/article/details/52507173)

从一个均匀分布[low,high)中随机采样，注意定义域是左闭右开，即包含low，不包含high.
- `low`: 采样下界，`float类型`，默认值为0；
- `high`: 采样上界，`float类型`，默认值为1；
- `size`: 输出样本数目，为int或元组(tuple)类型，例如，`size=(m,n,k)`, 则输出m*n*k个样本，缺省时输出1个值。
- 返回值：`ndarray`类型，其形状和参数`size`中描述一致。

类似`uniform`,还有以下随机数产生函数：
    a. `randint`: 原型：`numpy.random.randint(low, high=None, size=None, dtype='l')`，产生随机整数；
    b. `random_integers`: 原型： `numpy.random.random_integers(low, high=None, size=None)`，在闭区间上产生随机整数；
    c. `random_sample`: 原型： `numpy.random.random_sample(size=None)`，在[0.0,1.0)上随机采样；
    d. `random`: 原型： `numpy.random.random(size=None)`，和random_sample一样，是random_sample的别名；
    e. `rand`: 原型： `numpy.random.rand(d0, d1, ..., dn)`，产生d0 - d1 - ... - dn形状的在[0,1)上均匀分布的float型数。
    f. `randn`: 原型：`numpy.random.randn（d0,d1,...,dn)`,产生d0 - d1 - ... - dn形状的标准正态分布的float型数。

## np.random.seed(x…)
当我们设置相同的`seed`，每次生成的随机数相同，如果不指定`seed`，就是真随机数
```python
np.random.seed(0)
a=np.random.rand(4)
for var in range(5):
    print(a)
    np.random.seed(0)
    a = np.random.rand(4)
```
结果：
```
[0.5488135  0.71518937 0.60276338 0.54488318]
[0.5488135  0.71518937 0.60276338 0.54488318]
[0.5488135  0.71518937 0.60276338 0.54488318]
[0.5488135  0.71518937 0.60276338 0.54488318]
[0.5488135  0.71518937 0.60276338 0.54488318]
```

## np.random.permutation()	
返回一个随机排列
```python
[3 9 1 2 8 4 7 6 0 5]
```

## numpy.random.choice
`numpy.random.choice(a, size=None, replace=True, p=None)`

参数:
- a：一维数组或者int型变量，如果是数组，就按照里面的范围来进行采样，如果是单个变量，则采用`np.arange(a)`的形式
- size : int 或者 tuple of ints, 可选参数 (决定了输出的shape. 如果给定的是, (m, n, k), 那么 m * n * k 个采样点将会被采样. 默认为零，也就是只有一个采样点会被采样回来。)
- replace : 布尔参数，可选参数 (决定采样中是否有重复值)
- p :一维数组参数，可选参数 (对应着a中每个采样点的概率分布，如果没有标出，则使用标准分布。)

返回值: 
- samples : single item or ndarray

## np.argsort
`argsort(a, axis=-1, kind='quicksort', order=None)`
argsort函数返回的是数组值从小到大的索引值的列表。

```python
x = np.array([1, 4, 3, -1, 6, 9])
np.argsort(x)
# 输出定义为 y=array([3, 0, 2, 1, 4, 5])
# 我们发现argsort()函数是将x中的元素从小到大排列，提取其对应的index，然后输出
```

**np.argsort()[num]**
- 当num>=0时，np.argsort()[num]就可以理解为y[num];
- 当num<0时，np.argsort()[num]就是把数组y的元素反向输出，例如np.argsort()[-1]即输出x中最大值对应的index，np.argsort()[-2]即输出x中第二大值对应的index

# shape（属性）
返回元组，为对象的形状，若为一维`DataFrame`或`Series`则元组第二项维空(其实就是只有一个元素的元组)
例`(5,)`
 
# reshpae（方法）
是数组对象中的方法，用于改变数组的形状,也可以用来改变数据的维度，如1D->2D。

`reshape`函数生成的新数组和原始数组公用一个内存，也就是说，不管是改变新数组还是原始数组的元素，另一个数组也会随之改变：
 
**关于Python中reshape函数参数-1的意思？**

根据前值（或后值）来推测形状（可以用来偷懒）
 ```python
In [30]: obj = np.arange(25)

In [31]: obj.reshape(-1,5)
Out[31]:
array([[ 0,  1,  2,  3,  4],
       [ 5,  6,  7,  8,  9],
       [10, 11, 12, 13, 14],
       [15, 16, 17, 18, 19],
       [20, 21, 22, 23, 24]])
```
 
## numpy.mean() 
计算矩阵均值
```python
a = np.array([[1, 2], [3, 4]])  
np.mean(a) # 将上面二维矩阵的每个元素相加除以元素个数（求平均数）  
# 2.5
np.mean(a, axis=0) # axis=0，计算每一列的均值  
# array([ 2.,  3.])  
np.mean(a, axis=1) # 计算每一行的均值  
# array([ 1.5,  3.5]) 
```
 
## np.var（）
计算方差

```python
In [32]: np.var([6, 8, 10, 14, 18], ddof=1) # ddof 参数是贝塞尔矫正系数（无偏估计），求方差
Out[32]: 23.2
```
 
## np.cov（）
计算协方差

```python
>>> X=np.array([[1 ,5 ,6] ,[4 ,3 ,9 ],[ 4 ,2 ,9],[ 4 ,7 ,2]])
>>> np.cov(X)
array([[  7.  ,   4.5    ,       4.   ,   -0.5     ],
       [  4.5 ,  10.33333333,   11.5  ,  -7.16666667],
       [  4.  ,  11.5     ,      13.  ,   -8.5      ],
       [ -0.5 ,  -7.16666667,   -8.5  ,  6.33333333]])
```
 
## np.linalg.lstsq（） 
（本例返回两个相关因子和一个截距）
用最小二乘法拟合数据得到一个形如y = mx + c的线性方程

```python
In [34]: x = [[1,6,2],[1,8,1],[1,10,0],[1,14,2],[1,18,0]]

In [35]: y = [[7], [9], [13], [17.5], [18]]

In [36]: np.linalg.lstsq(x, y)[0]
C:\Users\dell\AppData\Local\Programs\Python\Python36\Scripts\ipython:1: FutureWarning: `rcond` parameter will change to the default of machine precision times ``max(M, N)`` where M and N are the input matrix dimensions.
To use the future default and silence this warning we advise to pass `rcond=None`, to keep using the old, explicitly pass `rcond=-1`.
Out[36]:
array([[1.1875    ],
       [1.01041667],
       [0.39583333]])
```
 
## numpy.linspace()
`numpy.linspace(start,stop,num=50,endpoint=True,retstep=False，dtype=None)`

`linspace`函数可以生成50个元素的等差数列。而前两个参数分别是数列的开头与结尾。第三个参数，可以制定数列的元素个数
```python
In [40]: np.linspace(1,50)
Out[40]:
array([ 1.,  2.,  3.,  4.,  5.,  6.,  7.,  8.,  9., 10., 11., 12., 13.,
       14., 15., 16., 17., 18., 19., 20., 21., 22., 23., 24., 25., 26.,
       27., 28., 29., 30., 31., 32., 33., 34., 35., 36., 37., 38., 39.,
       40., 41., 42., 43., 44., 45., 46., 47., 48., 49., 50.])
```

## np.logspace()
`numpy.logspace(start, stop, num=50, endpoint=True, base=10.0, dtype=None)[source]`
`logspac`用于创建等比数列，默认以10为底，元素个数是num

```python
[45]: np.logspace(0, 9, 10, base=2)
Out[45]: array([  1.,   2.,   4.,   8.,  16.,  32.,  64., 128., 256., 512.])
```

# min（）
返回最小值，可指定`axis`
```python
In [60]: c = np.array([[1,5,3],[4,2,6]])

In [61]: c.min()
Out[61]: 1

In [62]: c.min(axis=0)
Out[62]: array([1, 2, 3])

In [63]: c.min(axis=1)
Out[63]: array([1, 2])
```
 
# np.hstack() & np.vstack()
水平(按列顺序)把数组给堆叠起来
垂直（按列顺序）把数组给堆叠起来
```python
In [66]: arr1 = np.array([1, 2, 3])

In [67]: arr2 = np.array([4, 5, 6])

In [68]: np.hstack((arr1, arr2))
Out[68]: array([1, 2, 3, 4, 5, 6])

In [69]: np.vstack((arr1, arr2))
Out[69]:
array([[1, 2, 3],
       [4, 5, 6]])
```

## numpy.where（）
`numpy.where(condition[, x, y])`
1. 这里x,y是可选参数，condition是条件，这三个输入参数都是array_like的形式；而且三者的维度相同
2. 当conditon的某个位置的为true时，输出x的对应位置的元素，否则选择y对应位置的元素；
3. 如果只有参数condition，则函数返回为true的元素的坐标位置信息；

numpy.where()分两种调用方式：
1. 三个参数np.where(cond,x,y)：满足条件（cond）输出x，不满足输出y
```python
>>> aa = np.arange(10)
>>> np.where(aa,1,-1)
array([-1,  1,  1,  1,  1,  1,  1,  1,  1,  1])  # 0为False，所以第一个输出-1
```
2.	一个参数np.where(arry)：输出arry中‘真’值的坐标(‘真’也可以理解为非零)
```python
>>> a = np.array([2,4,6,8,10])
>>> np.where(a > 5)             # 返回索引
(array([2, 3, 4]),)   
```