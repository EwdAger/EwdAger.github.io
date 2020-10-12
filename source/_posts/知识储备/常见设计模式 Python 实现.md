---
title: 常见设计模式 Python 实现
tags:
  - 设计模式
  - Python
categories: 知识储备
abbrlink: 7798fba3
date: 2018-09-25 15:41:00
---

# 简单工厂模式
根据不同条件生产不同功能的类

```python
class op(object):
	def get_ans(self):
		pass

class Add(op):
	def get_ans(self):
		return self.a + self.b

class Mul(op):
	def get_ans(self):
		return self.a * self.b

class Undef(op):
	def get_ans(self):
		return "UNDEF!"

class Factory(op):
	operator = dict()
	operator["+"] = Add()
	operator["*"] = Mul()
	def create_operator(self, ch):
		t = self.operator[ch] if ch in self.operator else Undef()
		return t

if __name__ == "__main__":
	a = int(input())
	b = int(input())
	op = input()
	factory = Factory()
	cal = factory.create_operator(op)
	cal.a = a
	cal.b = b
	print(cal.get_ans())
```


# 单例模式

单例模式，也叫单子模式，是一种常用的软件设计模式。在应用这个模式时，单例对象的类 "类 (计算机科学)")必须保证只有一个实例存在。许多时候整个系统只需要拥有一个的全局对象，这样有利于我们协调系统整体的行为。

```python
# 非线程安全
class Singleton:

    _buff = {}

    def __init__(self, name):
        self.name = self.name if hasattr(self, 'name') else name

    def __new__(cls, *args, **kwargs):

        if cls not in cls._buff:
            instance = super().__new__(cls)
            cls._buff[cls] = instance
        return cls._buff[cls]


if __name__ == "__main__":
	s1 = Singleton("bar")
	s2 = Singleton("zoo")
	print(s1.name)
	print(s2.name)
```

```python
# 线程安全
from threading import Lock, Thread


class Singleton:
    _buff = {}
    _lock = Lock()

    def __init__(self, name):
        self.name = self.name if hasattr(self, 'name') else name

    def __new__(cls, *args, **kwargs):
        with cls._lock:
            if cls not in cls._buff:
                instance = super().__new__(cls)
                cls._buff[cls] = instance
        return cls._buff[cls]


def test_singleton(name):
    print(Singleton(name).name)


if __name__ == "__main__":
    process1 = Thread(target=test_singleton, args=("bar",))
    process2 = Thread(target=test_singleton, args=("zoo",))
    process1.start()
    process2.start()
```