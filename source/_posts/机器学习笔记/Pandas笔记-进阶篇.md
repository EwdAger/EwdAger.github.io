---
title: Pandas笔记-进阶篇
tags:
  - Python
  - Pandas
  - 数据分析
abbrlink: a61b68c3
categories: 机器学习笔记
date: 2019-08-27 13:56:00
---

# 汇总和计算描述统计

`panda`对象拥有一组常用的数学和统计方法，他们大部分都属于简约统计，NA值会自动被排除，除非通过`skipna=False`禁用

```python
In [78]: df
Out[78]:
    one  two
a  1.40  NaN
b  7.10 -4.5
c   NaN  NaN
d  0.75 -1.3

In [79]: df.sum()
Out[79]:
one    9.25
two   -5.80
dtype: float64

In [80]: df.sum(axis=1)
Out[80]:
a    1.40
b    2.60
c    0.00
d   -0.55
dtype: float64

In [81]: df.sum(skipna=False, axis=1)
Out[81]:
a     NaN
b    2.60
c     NaN
d   -0.55
dtype: float64
```



简约方法选项

选项 | 说明
--- | ---
axis | 简约的轴
skipna | 排除缺失值，默认True
level | 如果轴是层次化索引的，则根据level分组简约

描述和汇总统计

方法 | 说明
--- | ---
count | 非NA值的数量
describe | 针对Series或各DataFrame列计算汇总统计
min、max | 计算最小值和最大值
argmin、argmax | 计算能够获取到最小值和最大值的索引位置（整数）
idxmin、idxmax | 计算能够获取到最小值和最大值的索引值
quantile | 计算样本的分位数（0到1）
sum | 值的总和
mean | 值的平均数
median | 值的算术中位数（50%分位数）
mad | 根据平均值计算平均绝对离差
var | 样本值的方差
std | 样本值的标准差
skew | 样本值的偏度（三阶矩）
kurt | 样本值的峰度（四阶矩）
cumsum | 样本值的累计和
cummin、cummax | 样本值的累计最大值和累计最小值
cumprod | 样本值的累计积
diff | 计算一阶差分（对时间序列很有用）
pct_change | 计算百分数变化

## 相关系数与协方差

`corr`方法用于计算两个`Series`中重叠的、非NA的、按索引对齐的值的相关系数。`cov`方法用于计算协方差。

```python
In [18]: returns.cov()
Out[18]:
          AAPL       IBM      MSFT      GOOG
AAPL  0.001030  0.000254  0.000309  0.000303
IBM   0.000254  0.000369  0.000216  0.000142
MSFT  0.000309  0.000216  0.000516  0.000205
GOOG  0.000303  0.000142  0.000205  0.000580

In [19]: returns.corr()
Out[19]:
          AAPL       IBM      MSFT      GOOG
AAPL  1.000000  0.412392  0.423598  0.470676
IBM   0.412392  1.000000  0.494358  0.390689
MSFT  0.423598  0.494358  1.000000  0.443586
GOOG  0.470676  0.390689  0.443586  1.000000
```

看不太懂。。。留个笔记P146

## 唯一值、值计数以及成员资格

`unique`方法可以得到`Series`中唯一值的数据，返回的唯一值是未排序的。`value_counts`用于计算一个`Series`中各值出现的概率。`isin`方法计算表示`Series`各值是否包含传入的值序列中的布尔型数组。

```python
In [43]: s1 = Series(['c', 'c', 'b', 'b', 'a', 'd', 'e', 'f'])

In [45]: s1.unique()
Out[45]: array(['c', 'b', 'a', 'd', 'e', 'f'], dtype=object)

In [46]: pd.value_counts(s1)
Out[46]:
b    2
c    2
d    1
f    1
e    1
a    1
dtype: int64

In [52]: pd.value_counts(s1, sort=False)
Out[52]:
a    1
c    2
e    1
b    2
f    1
d    1
dtype: int64

In [54]: mask = s1.isin(['c', 'b'])

In [55]: mask
Out[55]:
0     True
1     True
2     True
3     True
4    False
5    False
6    False
7    False
dtype: bool
```

# 处理缺失数据

NA处理方法

方法 | 说明
--- | ---
dropna | 根据各标签的值中是否存在缺失数据对轴标签进行过滤，可通过阈值调节对缺失值的容忍度
fillna | 用指定值或插值方法（如ffill或bfill）填充缺失数据
isnull | 返回一个含有布尔值的对象，这些布尔值表示哪些值是缺失值/NA，改对象的类型与源类型一样
notnull | isnull的否定式

## 滤除缺失数据

对于`Series`很简单，只需要`dropna`可以轻松的滤除缺失数据，但在`DataFrame`中可以选择丢弃全NA或者含有NA的行或列。`dropna`默认丢弃任何含有缺失值的行。

```python
In [60]: data
Out[60]:
     0    1    2
0  1.0  6.5  3.0
1  1.0  NaN  NaN
2  NaN  NaN  NaN
3  NaN  6.5  3.0

In [61]: data.dropna()
Out[61]:
     0    1    2
0  1.0  6.5  3.0

In [62]: data
Out[62]:
     0    1    2
0  1.0  6.5  3.0
1  1.0  NaN  NaN
2  NaN  NaN  NaN
3  NaN  6.5  3.0

# 当限定的行或列全为NA时才滤除
In [63]: data.dropna(how='all')
Out[63]:
     0    1    2
0  1.0  6.5  3.0
1  1.0  NaN  NaN
3  NaN  6.5  3.0

In [64]: data.dropna(axis=1, how='all')
Out[64]:
     0    1    2
0  1.0  6.5  3.0
1  1.0  NaN  NaN
2  NaN  NaN  NaN
3  NaN  6.5  3.0
```

## 填充缺失数据

对于NA值，可以使用`fillna`方法，`fillna`方法默认返回新对象，但可以通过`inplace=True`参数原地修改。

```python
In [66]: data
Out[66]:
     0    1    2
0  1.0  6.5  3.0
1  1.0  NaN  NaN
2  NaN  NaN  NaN
3  NaN  6.5  3.0

In [67]: data2 = data

In [68]: data2.fillna(0, inplace=True)

In [69]: data2
Out[69]:
     0    1    2
0  1.0  6.5  3.0
1  1.0  0.0  0.0
2  0.0  0.0  0.0
3  0.0  6.5  3.0
```

`fillna`函数的参数

参数 | 说明
value | 用于填充缺失值的标量值或字典对象
method | 插值方式，如果函数调用时未指定其他参数的话，默认为"ffill"
axis | 待填充的轴，默认0
inplace | 修改调用者对象而不产生副本
limit | 可以连续填充的最大数量

# 层次化索引

层次化索引，是`pandas`可以在一个轴上拥有多个索引级别，它可以以低维度形式处理高维数据。
```python
In [71]: data = Series(np.random.randn(10), index=[['a','a','a','b','b','b','c'
    ...: ,'c','d','d'],[1,2,3,1,2,3,1,2,2,3]])

In [72]: data
Out[72]:
a  1    0.877453
   2    1.031048
   3   -0.585296
b  1    0.070123
   2   -0.988380
   3   -1.118804
c  1    0.963368
   2   -0.304830
d  2   -0.030440
   3    0.338128
dtype: float64

In [73]: data.a
Out[73]:
1    0.877453
2    1.031048
3   -0.585296
dtype: float64
```

## 重排分级顺序

```python
In [12]: frame
Out[12]:
state      Ohio     Colorado
color     Green Red    Green
key1 key2
a    1        0   1        2
     2        3   4        5
b    1        6   7        8
     2        9  10       11

In [13]: frame.swaplevel('key1', 'key2')
Out[13]:
state      Ohio     Colorado
color     Green Red    Green
key2 key1
1    a        0   1        2
2    a        3   4        5
1    b        6   7        8
2    b        9  10       11

```

## 根据级别汇总统计

```python
In [13]: frame
Out[13]:
state      Ohio     Colorado
color     Green Red    Green
key1 key2
a    1        0   1        2
     2        3   4        5
b    1        6   7        8
     2        9  10       11

In [14]: frame.sum(level='key2')
Out[14]:
state  Ohio     Colorado
color Green Red    Green
key2
1         6   8       10
2        12  14       16

In [15]: frame.sum(level='color', axis=1)
Out[15]:
color      Green  Red
key1 key2
a    1         2    1
     2         8    4
b    1        14    7
     2        20   10
```

## 使用DataFrame的列

`DataFrame`的`set_index`函数会将其一个或多个列转换成行索引，并创建一个新的`DataFrame`

```python
In [17]: frame
Out[17]:
   a  b    c  d
0  0  7  one  0
1  1  6  one  1
2  2  5  one  2
3  3  4  two  0
4  4  3  two  1
5  5  2  two  2
6  6  1  two  3

In [18]: frame2 = frame.set_index(['c','d'])

In [19]: frame2
Out[19]:
       a  b
c   d
one 0  0  7
    1  1  6
    2  2  5
two 0  3  4
    1  4  3
    2  5  2
    3  6  1
```

