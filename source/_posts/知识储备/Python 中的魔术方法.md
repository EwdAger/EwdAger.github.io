---
title: Python 中的魔术方法
tags:
  - Python
categories: 知识储备
abbrlink: 22aee029
date: 2018-09-25 18:39:00
---

# 构造与初始化

`__new__(self)`: 创建并返回一个类的实例，而`__init__`只是将传入的参数来初始化该实例，一般不需要重载`__new__`方法除非希望控制类的创建。

`__init__(self)`: 可以理解为构造函数，将传入的参数初始化成实例

`__del__(self)`: 可以理解为析构函数

# 属性访问控制

Python缺少对于类的封装,但人们希望Python能够定义私有属性，然后提供公共可访问的`getter`和 `setter`。 Python其实可以通过魔术方法来实现封装。

`__getattr__(self, name)`: 该方法定义了你试图访问一个不存在的属性时的行为。因此，重载该方法可以实现捕获错误拼写然后进行重定向, 或者对一些废弃的属性进行警告。
也可用于访问私有属性

`__setattr__(self, name, value)`: 是实现封装的解决方案，它定义了你对属性进行赋值和修改操作时的行为。
不管对象的某个属性是否存在,它都允许你为该属性进行赋值,因此你可以为属性的值进行自定义操作。有一点需要注意，实现`__setattr__`时要避免"无限递归"的错误。
```python
def __setattr__(self, name, value):
    self.name = value
    # 每一次属性赋值时, __setattr__都会被调用，因此不断调用自身导致无限递归了。
```

正确为：
```python
def __setattr__(self, name, value):
    self.__dict__[name] = value
```
