---
title: 经典排序算法 Python 实现
tags:
  - Python
categories: python学习心得&备忘
abbrlink: 576d991f
date: 2018-09-26 10:12:00
---

# 选择排序

每一趟遍历把最小的依次放在最前面，时间复杂度O(n²)

```python
def select_sort(n):
	for i in range(len(n):
		min = i
		for j in range(i, len(n)):
			if n[min] > n[j]:
				min = j
		if min != i:
			n[min], n[i] = n[i], n[min]
	return n
```
<!--more-->
# 冒泡排序

每一趟遍历把大的放前面，小的放后面，时间复杂度O(n²)

```python
def dubble_sort(n):
	for i in range(len(n)):
		for j in range(i+1, len(n)):
			if n[i] > n[j]:
				n[i], n[j] = n[j], n[i]
	return n
```

# 插入排序

在操作过程中维护一个排好序的片段，初始只包含一个元素。每次从未排序的片段取出一个元素插入正确的位置。时间复杂度为O(n²)

```python
def insert_sort(n):
	for i in (1, len(n)):
		x = n[i]
		j = i
		while j > 0 and n[j-1] > x:
			n[j] = n[j-1]
			j -= 1
		n[j] = x
	return n
```

# 快速排序

从数列中挑选出一个基准元素，把比基准小的放基准前，比基准大的放基准后。然后递归依次传入基准前后的序列。通常时间复杂度为O(nlogn)，最坏为O(n²)，当数列全相等、已经从小到大排好序或者从大到小排好序后为最坏情况

```python
def sub_sort(array,low,high):
    key = array[low]
    while low < high:
        while low < high and array[high] >= key:
            high -= 1
        while low < high and array[high] < key:
            array[low], array[high] = array[high], array[low]
            low += 1
    array[low] = key
    return low
 
 
def quick_sort(array,low,high):
     if low < high:
        key_index = sub_sort(array,low,high)
        quick_sort(array,low,key_index)
        quick_sort(array,key_index+1,high)

a = [3,4,6,8,2,1,5,8,9]
quick_sort(a, 0, len(a)-1)
```

To Be Contune