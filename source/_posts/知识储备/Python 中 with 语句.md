---
title: Python 中的上下文管理
tags:
  - Python
categories: 知识储备
abbrlink: c6f5fe28
date: 2020-09-29 10:48:00
---

# 0x00

当我们执行语句块前需要一些准备动作，在执行完成之后又需要执行一些收尾动作。比如：文件读写后需要关闭，数据库读写完毕后需要关闭连接，资源加锁解锁等情况。对于这种情况 python 提供了上下文管理的概念，可以通过上下文管理器处理代码块执行前的准备动作，以及执行后的收尾动作。

# 使用 with 语句

先来看看不使用上下文管理器的情况

```python
f = open("log.txt", "w")

try:
    f.write("hello")
finally:
    f.close()
```

使用上下文管理器

```python
with open("log.txt", "w") as f:
    f.write("hello")
```

当结束语句的时候，Python 会自动的帮我们调用 `f.close()`方法

## With 语句内部帮我们做了什么呢？

1. `with` 后面的 `open("log.txt", "w")` 语句返回对象的`__enter__`方法会被调用，并把`__enter__`的返回值赋值给`as`后面的变量

2. 当`with`执行完之后，会调用前端返回对象的`__exit__`方法。

让我们写个类测试一下：

```python
class ContextTest:
    def __enter__(self):
        print("进入enter！")
        return "Foo"

    def __exit__(self, *args, **kwargs):
        print("进入exit！")


def get_sample():
    return ContextTest()


with get_sample() as sample:
    print(f"Sample: {sample}")
```

运行结果：

```bash
进入enter！
Sample: Foo
进入exit！
```


# 自己实现一个上下文管理器

## 通过__enter__和__exit__实现

根据上面 with 语句的原理，我们自己使用类实现一个支持 with 语句的打开文件的类

```python
class File:
    def __init__(self, file_name: str, method: str):
        self.file_obj = open(file_name, method)

    def __enter__(self):
        return self.file_obj

    def __exit__(self, exc_type, exc_val, exc_tb):
        self.file_obj.close()
        print(f"type: {exc_type}")
        print(f"value: {exc_val}")
        print(f"traceback: {exc_tb}")


with File('demo.txt', "w") as f:
    f.useless_func()
```

我们这里还实现了一个异常的处理，关于异常处理，with 语句会这样执行

1. 它把异常的`exc_type`, `exc_val`, `exc_tb`传递`__exit__`方法
2. 它让`__exit__`方法来处理异常 
3. 如果`__exit__`返回的是True，那么这个异常就被忽略。
4. 如果`__exit__`返回的是True以外的任何东西，那么这个异常将被with语句抛出。

输出的结果为：

```bash
type: <class 'AttributeError'>
value: '_io.TextIOWrapper' object has no attribute 'useless_func'
traceback: <traceback object at 0x000001D259D82908>
Traceback (most recent call last):
  ...
AttributeError: '_io.TextIOWrapper' object has no attribute 'useless_func'
```


## 通过contexlib模块装饰器和生成器实现

```python
from contextlib import contextmanager


@contextmanager
def my_open(filename, mode):
    file_obj = open(filename, mode)
    try:
        yield file_obj.readlines()
    except Exception as e:
        raise e

    finally:
        file_obj.close()


if __name__ == '__main__':
    with my_open(r'demo.txt', 'r') as f:
        for line in f:
            print(line)
```

yield之前的代码由`__enter__`方法执行，yield之后的代码由`__exit__`方法执行。本质上还是`__enter__`和`__exit__`方法。因为`@contextmanager`这个decorator接受一个generator，用`yield`语句把`with ... as var`把变量输出出去，然后，with 语句就可以正常地工作了

# contextlib 模块

除了接管文件、数据库等的打开关闭，我们还可以用`@contextmanager`的特性做一些很棒的事情，如果我们希望在某段代码执行前后自动执行特定代码，也可以使用`@contextmanager`实现

```python
@contextmanager
def tag(name):
    print("<%s>" % name)
    yield
    print("</%s>" % name)

with tag("h1"):
    print("hello")
    print("world")

# 输出
<h1>
hello
world
</h1>
```

## @closing

如果一个对象没有实现上下文，我们就不能把它用于`with`语句，这个时候我们可以用`closing`把对象变成上下文对象。例如，用`with`语句使用`urlopen()`

```python
from contextlib import closing
from urllib.request import urlopen

with closing(urlopen('https://www.python.org')) as page:
    for line in page:
        print(line)
```

让我们看看这个神奇的`closing`的源码

```python
class closing(AbstractContextManager):
    """Context to automatically close something at the end of a block.

    Code like this:

        with closing(<module>.open(<arguments>)) as f:
            <block>

    is equivalent to this:

        f = <module>.open(<arguments>)
        try:
            <block>
        finally:
            f.close()

    """
    def __init__(self, thing):
        self.thing = thing
    def __enter__(self):
        return self.thing
    def __exit__(self, *exc_info):
        self.thing.close()
```

就跟我们在上面用类方法实现的一样嘛~


# 参考文献
[廖雪峰的Python教程 - contextlib](https://www.liaoxuefeng.com/wiki/1016959663602400/1115615597164000)
[python with语句上下文管理的两种实现方法](https://www.cnblogs.com/tkqasn/p/6003959.html)
[Python 中 with用法及原理](https://blog.csdn.net/u012609509/article/details/72911564)