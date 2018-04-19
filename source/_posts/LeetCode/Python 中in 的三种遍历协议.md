---
title: Python中in的三种遍历协议
tags:
  - LeetCode
  - python
categories: LeetCode刷题总结
abbrlink: b7cb2971
date: 2018-04-18 21:23:00
---

今天刷LeetCode碰到个简单题 [宝石与石头](https://leetcode-cn.com/problems/jewels-and-stones/description/)想降一下时间复杂度，就没有用`in` list写法，直接又遍历了一遍列表，可没想到一看答案，这样做的用时比用`in`要慢得多。索性研究了一下`in`的原理。这里复制一下知乎用户（很抱歉， 这位同学账号似乎被封了，无法著名）的答案。

`in` 关键字实现了一套python中的遍历协议.

## 协议A: __iter__ + next
> 循环时,  程序先使用__iter__ (相当于iter(instance))获取具有next方法的对象, 然后通过其返回的对象, 不断调用其next方法, 直到StopIteration错误抛出.


```python
class A:
    def __iter__(self):
        self.limit = 4
        self.times = 0
        self.init = 1
        return self

    def next(self):
        if self.times >= self.limit:
            raise StopIteration()
        else:
            x = self.init
            self.times += 1
            self.init += 1
            return x

print 'A>>>>>>'
for x in A():
    print x
```

<!--more-->
打印结果：


```python
A>>>>>>
1
2
3
4
```


## 协议B: __getitem__ + __len__
> 循环时, 程序先调用__len__ (相当于len(instance))方法获取长度, 然后循环使用 __getitem__(index) (相当于instance[index])获取元素, index in range(len(instance))


```python
class B:

    def __init__(self):
        self._list = [5, 6, 7, 8]

    def __getitem__(self, slice):
        return self._list[slice]

    def __len__(self):
        return len(self._list)

print 'B>>>>>>'
for x in B():
    print x
```

打印结果：

```python

B>>>>>>
5
6
7
8
```

## 协议C: yield关键字

```python
def C():
    for x in range(9, 13):
        yield x

print 'C>>>>>>'
for x in C():
    print x
```

打印结果：

```python
C>>>>>>
9
10
11
12
```

从上边可以看出, ABC三种方式都可以实现in的循环, 对于A和B, 如果一个类把这两个方案都实现了怎么办?

```python
class D(A, B):
    pass
print 'D>>>>>>'
for x in D():
    print x
```

打印结果：

```python
D>>>>>>
1
2
3
4
```

可见, in优先使用的是A计划.


> 作者：知乎用户
> 链接：https://www.zhihu.com/question/24868112/answer/83471042
> 来源：知乎
> 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。