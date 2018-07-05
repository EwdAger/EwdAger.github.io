---
title: Python 中*的特殊用法(解包元组)
tags:
  - LeetCode
categories: LeetCode刷题总结
abbrlink: f52d4c66
date: 2018-07-05 11:32:00
---

## unpack tuples

要将一个`tuple`传入一个函数中，必须先解包成单独的数，才能使用，比如

```python
...
>>> brith = (2018-7-5)
>>> datetime.date(brith)

Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: an integer is required (got type tuple)

```

所以正确的应该是

```python
>>> brith = (2018-7-5)
>>> datetime.date(*brith) # 注意*号
```

来源: [what-is-the-pythonic-way-to-unpack-tuples](https://stackoverflow.com/questions/2238355/what-is-the-pythonic-way-to-unpack-tuples)