---
title: Python异常处理traceback和exc_info
tags:
  - Python
  - 异常处理
categories: python学习心得&备忘
abbrlink: 1a6c337e
date: 2019-10-11 17:29:00
---


开发过程中一般都会使用`traceback`将捕获到的异常打印出来。

```python
import traceback

def fake_exception():
    1 / 0

def catch_exception():
    try:
        fake_exception()
    except:
        traceback.print_exc()

catch_exception()
```
结果
```python
Traceback (most recent call last):
  File ".\test.py", line 9, in catch_exception
    fake_exception()
  File ".\test.py", line 5, in fake_exception
    1 / 0
ZeroDivisionError: integer division or modulo by zero
```



事实上，`traceback`里的所有信息都是从`exc_info`里面获取的。
```
traceback.print_exc([limit[, file]])
In fact, it uses sys.exc_info() to retrieve the same information in a thread-safe way instead of using the deprecated variables.
```

那么我们再来看一下`exc_info()`这个方法。
https://docs.python.org/2/library/sys.html?highlight=sys#module-sys
该方法返回三个值：`type`, `value`, `traceback`.

- **type** (异常类别)
    get the exception type of the exception being handled (a class object)
- **value** (异常说明，可带参数)
    get the exception parameter (a class instance)
- **traceback** (traceback对象，包含更丰富的信息)
    get a traceback object which encapsulates the call stack at the point where the exception originally occurred (a traceback object)

其中`traceback`中还包含了更为丰富的信息，比如文件名，行号等等。如果觉得系统默认的`traceback`打印格式不好看的话，可以利用`exc_info`的返回值自定义格式。

```python
import sys

def fake_exception():
    1 / 0

def catch_exception():
    try:
        fake_exception()
    except:
        e_type, e_value, e_traceback = sys.exc_info()
        print "type ==> %s" % (e_type.__name__)
        print "value ==> %s" %(e_value)
        print "traceback ==> file name: %s" %(e_traceback.tb_frame.f_code.co_filename)
        print "traceback ==> line no: %s" %(e_traceback.tb_lineno)
        print "traceback ==> function name: %s" %(e_traceback.tb_frame.f_code.co_name)

catch_exception()
```
结果显示
```python
type ==> typename: ZeroDivisionError
value ==> message: integer division or modulo by zero
traceback ==> fielname: .\test.py
traceback ==> lineno: 8
traceback ==> name: catch_exception
```

[原文地址](https://www.jianshu.com/p/b342b19657fc)