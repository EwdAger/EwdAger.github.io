---
title: Python 单链表实现&基础函数
tags:
  - Python3
  - Python
  - 编码
categories: python学习心得&备忘
abbrlink: 2870634a
date: 2018-09-09 11:21:00
---

前两天面滴滴，被问到怎么判断两个链表是否相交，然后并不懂什么是单链表相交...就很尴尬。 赶紧复习一下单链表的知识。


# 单链表实现

```python
class LNode:
	def __init__(self, elem, next_ = None):
		self.elem = elem
		self.next = next_

class LList:
	def __init__(self):
		self._head = None

	def is_empty(self):
		return self._head is None

	# 前端插入
	def prepend(self, elem):
		self._head = LNode(elem, self._head)

	# 删除头结点并返回
	def pop_first(self):
		if self.is_empty():
			raise LinkedListUnderFlow("in pop")
		e = self._head.elem
		self._head = self._head.next
		return e

	def append(self, elem):
		if self.is_empty():
			self._head = LNode(elem)
			return
		p = self._head
		while p.next:
			p = p.next
		p.next = LNode(elem)

	def pop(self):
		if self.is_empty():
			raise LinkedListUnderFlow("in pop")
		p = self._head
		if p.next is None:
			e = p.elem
			self._head = None
			return e

		# 用p.next.next做条件是因为把最后一个结点删除，需要找到倒数第二个结点
		while p.next.next is not None:
			p = p.next
		e = p.next.elem
		p.next = None
		return e

	def find(self, pred):
		p = self._head
		while p.next is not None:
			if pred(p.elem):
				# 构建生成器，找到了一个元素可以继续找下一个
				yield p.elem
			p = p.next


	def printall(self):
		p = self._head
		while p is not None:
			print(p.elem, end="")
			if p.next is not None:
				print(", ", end="")
			p = p.next

		# 换行，因为默认end参数为 "\n"
		# 等价 print("\n", end="")
		print("")

if __name__ == '__main__':
	my_list = LList()
	for i in range(10, 0, -1):
		my_list.prepend(i)
	my_list.printall()    # 1, 2, 3, 4, 5, 6, 7, 8, 9, 10
	for i in range(11, 21):
		my_list.append(i)
	my_list.printall()
	print(my_list.pop())  # 20
	print(my_list.pop_first()) # 1
	my_list.printall()   # 2~19

	def find_odd(n):
		if n % 2 != 0:
			return n
	for i in my_list.find(find_odd):
		print(i)         # 1, 3, 5, 7, 9 ,...
```

# 补充一下print的参数
- print(value, ..., sep=' ', end='\n', file=sys.stdout, flush=False)
- sep: 多个参数之间的分隔字符串
- end: print结束后的字符串
- file: 输出到**已打开的文件**，注意，当文件关闭后才会保存
- flush: 所有数据打印到控制台，立即“刷新”到实际控制台并保留待处理的打印缓冲区
  可用于上面的文件操作，当文件未关闭时及时输出到控制台，参考[Stack Overflow](https://stackoverflow.com/questions/15608229/what-does-prints-flush-do)