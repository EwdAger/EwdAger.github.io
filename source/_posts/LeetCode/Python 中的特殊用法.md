---
title: Python 元组解包的几种方法
tags:
  - LeetCode
categories: LeetCode刷题总结
abbrlink: bba523e0
date: 2018-07-05 11:32:00
---

## 访问下标解包

这其实都不算解包了吧。。

```python
>>> a = (1,2,3)
>>> a[0]
1
```

## 赋值解包

```python
>>> a = (1,2,3,)
>>> b, c, d = a
```

## 星号(\*)解包

要将一个`tuple`中的所有值作为参数，如果直接用上面两种方法就不太 pythonic了，可以用以下方法解包

```python
...
>>> brith = (2018, 7, 5,)
>>> datetime.date(brith)     # 当然这里直接传入元组是不行的，该函数要求传入int类型

Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: an integer is required (got type tuple)

```

所以正确的应该是

```python
>>> import datetime
>>> brith = (2018, 7, 5,)
>>> print(datetime.date(*brith))  # 注意*号
2018-7-5
```

来源: [what-is-the-pythonic-way-to-unpack-tuples](https://stackoverflow.com/questions/2238355/what-is-the-pythonic-way-to-unpack-tuples)