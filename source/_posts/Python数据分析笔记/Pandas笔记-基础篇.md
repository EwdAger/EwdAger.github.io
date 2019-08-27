---
title: Pandas笔记-基础篇
tags:
  - Python
  - Pandas
  - 数据分析
abbrlink: 8493c310
categories: Python数据分析笔记
date: 2019-08-26 09:59:00
---

# Series

`Series`是一种类似一维数组的对象，由一组数据和一组与之相关的数据索引组成

```python
In [9]: obj = Series([4,7,-5,3])

In [10]: obj.index
Out[10]: RangeIndex(start=0, stop=4, step=1)

In [11]: obj.values
Out[11]: array([ 4,  7, -5,  3], dtype=int64)

In [12]: obj2 = Series([4,7,-5,3], index=['d','b','c','a'])

In [13]: obj2.index
Out[13]: Index(['d', 'b', 'c', 'a'], dtype='object')

In [14]: obj2['a']
Out[14]: 3

In [15]: obj2[['c','b','a']]
Out[15]:
c   -5
b    7
a    3
dtype: int64

```
<!--more-->
Numpy 数组运算都会保留索引和值之间的链接，但这些操作并不会改变原`Series`本身（与`ndarray`的选区操作相对）

```python
In [19]: obj2[obj2>0]
Out[19]:
d    4
b    7
a    3
dtype: int64

In [20]: obj2 * 2
Out[20]:
d     8
b    14
c   -10
a     6
dtype: int64

In [23]: np.exp(obj2)
Out[23]:
d      54.598150
b    1096.633158
c       0.006738
a      20.085537
dtype: float64

```

还可以将`Series`看成一个定长的有序字典，因为它是索引值到数据值的一个映射。它可以用在许多原本需要的字典参数函数中。

```python
In [26]: obj2
Out[26]:
d    4
b    7
c   -5
a    3
dtype: int64

In [27]: 'd' in obj2
Out[27]: True

In [28]: 4 in obj2
Out[28]: False

In [29]: 4 in obj2.values
Out[29]: True
```

所有数据被放在一个Python字典中时，也可以直接用这个字典来创建`Series`。如果只传入一个字典，结果`Series`的索引就是原字典的键（有序排列）。

```python
In [30]: sdata = {'Ohio': 3500, 'Texas': 71000, 'Oregon': 16000, 'Utah':5000}

In [31]: obj3 = Series(sdata)

In [32]: obj3
Out[32]:
Ohio       3500
Texas     71000
Oregon    16000
Utah       5000
dtype: int64

In [33]: status = ['California', 'Ohio', 'Oregon', 'Texas']

# 跟索引相匹配的3个值会被找出来并放到相应的位置是，如果找不到值就为NaN
In [34]: obj4 = Series(sdata, index=status)

In [35]: obj4
Out[35]:
California        NaN
Ohio           3500.0
Oregon        16000.0
Texas         71000.0
dtype: float64
```

`Series`可用`pd.isnull`或`obj4.isnull()`两种方式判断是否为空

```python
In [36]: pd.isnull(obj4)
Out[36]:
California     True
Ohio          False
Oregon        False
Texas         False
dtype: bool

In [37]: pd.notnull(obj4)
Out[37]:
California    False
Ohio           True
Oregon         True
Texas          True
dtype: bool

In [38]: obj4.isnull()
Out[38]:
California     True
Ohio          False
Oregon        False
Texas         False
dtype: bool
```

`Series`还有个重要的特征就是进行数学运算的时候会自动对齐不同索引的数据。

```python
In [41]: obj4+obj3
Out[41]:
California         NaN
Ohio            7000.0
Oregon         32000.0
Texas         142000.0
Utah               NaN
dtype: float64
```

`Series`对象本身和索引都有一个`Name`属性，`Series`的索引可以通过赋值的方式修改。

```python
In [42]: obj4
Out[42]:
California        NaN
Ohio           3500.0
Oregon        16000.0
Texas         71000.0
dtype: float64

In [43]: obj4.name = 'population'

In [44]: obj4.index.name = 'state'

In [45]: obj4
Out[45]:
state
California        NaN
Ohio           3500.0
Oregon        16000.0
Texas         71000.0
Name: population, dtype: float64

In [46]: obj.index
Out[46]: RangeIndex(start=0, stop=4, step=1)

In [47]: obj.index = ['a', 'b', 'c', 'd']

In [48]: obj
Out[48]:
a    4
b    7
c   -5
d    3
dtype: int64
```

# DataFrame

`DataFrame`可以被看做有`Series`组成的字典（共用一个索引），构建`DataFrame`最常用的方法是直接传入一个由等长列表或Numpy数组组成的字典。

```python
In [49]: data = {'state': ['Ohio', 'Ohio', 'Ohio', 'Nevada', 'Nevada'],
    ...: 'year': [2000, 2001, 2002, 2001, 2002],
    ...: 'pop': [1.5, 1.7, 3.6, 2.4, 2.9]}

In [50]: frame = DataFrame(data)

In [51]: frame
Out[51]:
    state  year  pop
0    Ohio  2000  1.5
1    Ohio  2001  1.7
2    Ohio  2002  3.6
3  Nevada  2001  2.4
4  Nevada  2002  2.9
```

当传入的列在数据中找不到时，就会产生NA值。`DataFrame`还可以通过类似字典标记的方式或属性的方式获取`DataFrame`为一个`Series`。*但使用属性的方式有可能与预留方法名重名，推荐使用字典标记方式*

```python
In [53]: frame2 = DataFrame(data, columns=['year', 'state', 'pop', 'debt'], ind
    ...: ex=['one', 'two', 'three', 'four', 'five'
    ...: ])

In [54]: frame2
Out[54]:
       year   state  pop debt
one    2000    Ohio  1.5  NaN
two    2001    Ohio  1.7  NaN
three  2002    Ohio  3.6  NaN
four   2001  Nevada  2.4  NaN
five   2002  Nevada  2.9  NaN

In [57]: frame2.state
Out[57]:
one        Ohio
two        Ohio
three      Ohio
four     Nevada
five     Nevada
Name: state, dtype: object

In [58]: frame2['state']
Out[58]:
one        Ohio
two        Ohio
three      Ohio
four     Nevada
five     Nevada
Name: state, dtype: object
```

`DataFrame`支持给某一列附上一个标量的值或者一组值。将列表或者数组赋值给某个列时，其长度必须与`DataFrame`长度匹配。如果赋值的是`Series`，就会精准匹配`DataFrame`的索引，所有空位都将被填上缺失值。

```python
In [75]: frame2['debt'] = 1

In [76]: frame2
Out[76]:
       year   state  pop  debt
one    2000    Ohio  1.5     1
two    2001    Ohio  1.7     1
three  2002    Ohio  3.6     1
four   2001  Nevada  2.4     1
five   2002  Nevada  2.9     1

In [77]: frame2['debt'] = np.arange(5.)

In [78]: frame2
Out[78]:
       year   state  pop  debt
one    2000    Ohio  1.5   0.0
two    2001    Ohio  1.7   1.0
three  2002    Ohio  3.6   2.0
four   2001  Nevada  2.4   3.0
five   2002  Nevada  2.9   4.0
```

## 索引对象

`DataFrame`的`index`是不可以直接修改的，即`frame.index[1] = 5`是不可取的。以下提供了`index`的方法和属性

方法 | 说明
--- | ---
append | 链接另一个index对象，产生一个新的index
diff | 计算差集，并得到一个index
intersection | 计算交集
union | 计算并集
isin | 计算一个指示各值是否都包含在参数集合中的布尔型数组
delete | 删除索引i处的元素，并的到新的index
drop | 删除传入的值，并得到新的index
insert | 将元素插入到索引i处，并得到新的index
is_monotonic | 当个元素均大于等于前一个元素时，返回True
is_union | 当index没有重复值时，返回True
unique | 计算index中唯一值得数组

# 基本功能

## 重新索引

`reindex`可以创建一个适应新索引的新对象。

```python
In [86]: obj = Series([4.5, 7.2, -5.3, 3.6], index=['d', 'b', 'a', 'c'])

In [87]: obj
Out[87]:
d    4.5
b    7.2
a   -5.3
c    3.6
dtype: float64

In [88]: obj.reindex(['a', 'b', 'c', 'd', 'e'], fill_value=0)
Out[88]:
a   -5.3
b    7.2
c    3.6
d    4.5
e    0.0
dtype: float64
```

`reindex`重新索引时还可以做一些插值处理，`method`选项即可达到此目的，例如`ffill`就可以实现向前填充值。

```python
In [90]: obj3
Out[90]:
0      blue
2    purple
4    yellow
dtype: object

In [91]: obj3.reindex(range(6), method='ffill')
Out[91]:
0      blue
1      blue
2    purple
3    purple
4    yellow
5    yellow
dtype: object
```

`reindex`中可用的`method`选项如下：

参数 | 说明
--- | ---
ffill、pad | 向前填充（或搬运）值
bfill、backfill | 向后填充（或搬运）值

`reindex`函数的参数
参数 | 说明
index | 用作索引的新序列
method | 插值（填充）方式
fill_value | 在重新索引过程中，需要引入缺失值时使用的替代值
limit | 向前或向后填充时的最大值
level | 在MultiIndex的指定级别上匹配简单索引，否则选取其子集
copy | 默认True，无论何时都复制；如果为False，则新旧相等就不复制

## 丢弃制定轴上的项

使用`drop`方法可以丢弃某条轴上一个或多个项

```python
In [94]: frame.drop(0)
Out[94]:
    state  year  pop
1    Ohio  2001  1.7
2    Ohio  2002  3.6
3  Nevada  2001  2.4
4  Nevada  2002  2.9

In [95]: frame.drop('state', axis=1)
Out[95]:
   year  pop
0  2000  1.5
1  2001  1.7
2  2002  3.6
3  2001  2.4
4  2002  2.9
```

## 索引、选取和过滤

类型 | 说明
--- | ---
obj[val] | 选取DataFrame的单个列或一组列，在一些特殊情况下回比较便利：布尔型数组（过滤行）、切片（行切片）、布尔型DataFrame（根据条件设置值）
obj.ix[val] | 选取DataFrame的单个行或一组行
obj.ix[:, val] | 选取单个列或列子集
obj.ix[val1, val2] | 同时选取行和列
reindex方法 | 将一个或多个轴匹配到新索引
xs方法 | 根据标签选取单行或单列，并返回一个Series
icol、irow | 根据整数位置选取单列或单行，并返回一个Series
get_value、set_value方法 | 根据行标签和列标签选取单个值

## 算术运算和数据对齐

`pandas`最重要的一个功能是，它可以对不同索引的对象进行算术运算。在将对象相加时，如果存在不同的索引，则结果的索引就是该索引对的并集。自动的数据对齐操作在不重叠的索引处引入了NA值。

## 在算术方法中填充值

不使用`+`可以使用`add`方法进行相加，其中可以添加`fill_value`参数填充索引不重叠产生的缺省值。

```python
In [14]: df1
Out[14]:
     a    b     c     d
0  0.0  1.0   2.0   3.0
1  4.0  5.0   6.0   7.0
2  8.0  9.0  10.0  11.0

In [15]: df2
Out[15]:
      a     b     c     d     e
0   0.0   1.0   2.0   3.0   4.0
1   5.0   6.0   7.0   8.0   9.0
2  10.0  11.0  12.0  13.0  14.0
3  15.0  16.0  17.0  18.0  19.0

In [16]: df1.add(df2, fill_value=0)
Out[16]:
      a     b     c     d     e
0   0.0   2.0   4.0   6.0   4.0
1   9.0  11.0  13.0  15.0   9.0
2  18.0  20.0  22.0  24.0  14.0
3  15.0  16.0  17.0  18.0  19.0
```

灵活的算术方法

方法 | 说明
--- | ---
add | 加法
sub | 减法
div | 除法
mul | 乘法

## DataFrame和Series之间的运算

默认情况下，`DataFrame`和`Series`之间的算术运算会将Series的索引匹配到`DataFrame`的列然后沿着行一直向下广播：

```python
In [17]: df1
Out[17]:
     a    b     c     d
0  0.0  1.0   2.0   3.0
1  4.0  5.0   6.0   7.0
2  8.0  9.0  10.0  11.0

In [19]: s1 = df1.iloc[1]

In [20]: s1
Out[20]:
a    4.0
b    5.0
c    6.0
d    7.0
Name: 1, dtype: float64

In [21]: df1 - s1
Out[21]:
     a    b    c    d
0 -4.0 -4.0 -4.0 -4.0
1  0.0  0.0  0.0  0.0
2  4.0  4.0  4.0  4.0
```

如果希望匹配行并在列上广播，则必须使用算术运算方法

```python
In [24]: df1
Out[24]:
     a    b     c     d
0  0.0  1.0   2.0   3.0
1  4.0  5.0   6.0   7.0
2  8.0  9.0  10.0  11.0

In [25]: s1 = Series([1,2,3], index=[0,1,2])

In [26]: s1
Out[26]:
0    1
1    2
2    3
dtype: int64

In [27]: df1.add(s1, axis=0)
Out[27]:
      a     b     c     d
0   1.0   2.0   3.0   4.0
1   6.0   7.0   8.0   9.0
2  11.0  12.0  13.0  14.0
```

## 函数应用和映射

1. `Numpy`的`ufuncs`也可用于操作`pandas`对象
2. 也可用`DataFrame`的`apply`方法实现

```python
In [29]: frame
Out[29]:
               b         c         d
Utah   -0.677682  0.247555  0.145010
Ohio   -0.120664  0.618269  1.170003
Texas   0.692635 -1.465877  0.432430
Oregon -0.103928  1.032560  1.944111

# Numpy的ufuncs
In [30]: np.abs(frame)
Out[30]:
               b         c         d
Utah    0.677682  0.247555  0.145010
Ohio    0.120664  0.618269  1.170003
Texas   0.692635  1.465877  0.432430
Oregon  0.103928  1.032560  1.944111

In [31]: f = lambda x: x.max() - x.min()

# apply方法
In [32]: frame.apply(f)
Out[32]:
b    1.370317
c    2.498437
d    1.799101
dtype: float64

In [33]: frame.apply(f, axis=1)
Out[33]:
Utah      0.925237
Ohio      1.290666
Texas     2.158512
Oregon    2.048038
dtype: float64
```

处理元素级的Python函数，可以使用`applymap`(对于DataFrame)，对于`Series`可以使用`map`方法

```python
In [34]: f2 = lambda x: '%.2f' % x

# 对DataFrame
In [35]: frame.applymap(f2)
Out[35]:
            b      c     d
Utah    -0.68   0.25  0.15
Ohio    -0.12   0.62  1.17
Texas    0.69  -1.47  0.43
Oregon  -0.10   1.03  1.94

# 对Series
In [36]: frame['b'].map(f2)
Out[36]:
Utah      -0.68
Ohio      -0.12
Texas      0.69
Oregon    -0.10
Name: b, dtype: object

```

## 排名和排序

可用`sort_index`方法进行排序，其中可以设定轴与降序升序

```python
In [42]: frame
Out[42]:
       d  a  b  c
three  0  1  2  3
one    4  5  6  7

In [43]: frame.sort_index()
Out[43]:
       d  a  b  c
one    4  5  6  7
three  0  1  2  3

In [44]: frame.sort_index(axis=1)
Out[44]:
       a  b  c  d
three  1  2  3  0
one    5  6  7  4

# 降序排序，默认ascending=True，为升序
In [45]: frame.sort_index(axis=1, ascending=False)
Out[45]:
       d  c  b  a
three  0  3  2  1
one    4  7  6  5
```

`rank`函数可以为各组分配一个平均排名，默认是排名越高数值越大(ascending=True)

```python
In [65]: obj
Out[65]:
0    7
1   -5
2    7
3    4
4    3
5    0
6    4
dtype: int64

In [66]: obj.rank()
Out[66]:
0    6.5
1    1.0
2    6.5
3    4.5
4    3.0
5    2.0
6    4.5
dtype: float64

# 降序排名，因为7，4出现了两次，所以排名带有.5
In [67]: obj.rank(ascending=False)
Out[67]:
0    1.5
1    7.0
2    1.5
3    3.5
4    5.0
5    6.0
6    3.5
dtype: float64

In [68]: obj.rank(method='first', ascending=False)
Out[68]:
0    1.0
1    7.0
2    2.0
3    3.0
4    5.0
5    6.0
6    4.0
dtype: float64
```

排名时用于破坏平级关系的method选项

method | 说明
--- | ---
average | 默认：在相等分组中，为各个值分配平均排名
min | 使用整个分组的最小排名
max | 使用整个分组的最大排名
first | 按值在原始数据中的出现顺序分配排名

## 带有重复值得轴索引

虽然许多`pandas`函数如`reindex`都要求标签唯一，但这并不是强制性的。

```python
In [69]: obj = Series(range(5), index=['a', 'a', 'b', 'c', 'c'])

In [70]: obj
Out[70]:
a    0
a    1
b    2
c    3
c    4
dtype: int64

In [71]: obj['a']
Out[71]:
a    0
a    1
dtype: int64

In [72]: obj.is_unique
Out[72]: True

In [73]: obj.index.is_unique
Out[73]: False

In [74]: obj[0] = 1

In [75]: obj
Out[75]:
a    1
a    1
b    2
c    3
c    4
dtype: int64

In [76]: obj.is_unique
Out[76]: False
```