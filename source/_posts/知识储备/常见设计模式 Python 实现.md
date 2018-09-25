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

<!--more-->
# 单例模式

单例模式，也叫单子模式，是一种常用的软件设计模式。在应用这个模式时，单例对象的类 "类 (计算机科学)")必须保证只有一个实例存在。许多时候整个系统只需要拥有一个的全局对象，这样有利于我们协调系统整体的行为。

```python
class Singleton:
	class __My_class:
		def __init__(self, arg):
			self.arg = arg

		def show(self):
			return (id(self), self.arg)

	_instance = None
	def __init__(self, arg):
		if not Singleton._instance:
			Singleton._instance = Singleton.__My_class(arg)
		else:
			Singleton._instance.arg = arg
	def __getattr__(self, attr):
		return getattr(self._instance, attr)

if __name__ == "__main__":
	s1 = Singleton("bar")
	s2 = Singleton("zoo")
	print(s1.show())
	print(s2.show())
```