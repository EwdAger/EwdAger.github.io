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
from typing import List, Optional

NumsList = Optional[List[int]]


class SelectSort:

    @staticmethod
    def select_sort(nums: NumsList) -> NumsList:

        for i in range(len(nums)):
            min_idx = i
            for j in range(i, len(nums)):
                if nums[j] < nums[min_idx]:
                    min_idx = j
            nums[i], nums[min_idx] = nums[min_idx], nums[i]

        return nums


if __name__ == "__main__":
    nums = [2, 4, 5, 1, 2, 9, 10, 20]
    print(SelectSort().select_sort(nums))
```

# 冒泡排序

每一趟遍历把大的放前面，小的放后面，时间复杂度O(n²)

```python
from typing import List, Optional

NumsList = Optional[List[int]]


class BobbleSort:

    @staticmethod
    def bobble_sort(nums: NumsList) -> NumsList:
        for i in range(len(nums)):
            for j in range(i+1, len(nums)):
                if nums[i] > nums[j]:
                    nums[i], nums[j] = nums[j], nums[i]

        return nums


if __name__ == "__main__":
    nums = [2, 4, 5, 1, 2, 9, 10, 20]
    print(BobbleSort().bobble_sort(nums))
```

# 插入排序

在操作过程中维护一个排好序的片段，初始只包含一个元素。每次从未排序的片段取出一个元素插入正确的位置。时间复杂度为O(n²)

```python
from typing import List, Optional

NumsList = Optional[List[int]]


class InsertSort:
    @staticmethod
    def insert_sort(nums: NumsList) -> NumsList:
        for i in range(1, len(nums)):
            target = nums[i]

            j = i - 1
            while j >= 0 and target < nums[j]:
                nums[j + 1] = nums[j]
                j -= 1
            nums[j + 1] = target

        return nums


if __name__ == "__main__":
    nums = [2, 4, 5, 1, 2, 9, 10, 20]
    print(InsertSort().insert_sort(nums))
```

# 快速排序

从数列中挑选出一个基准元素，把比基准小的放基准前，比基准大的放基准后。然后递归依次传入基准前后的序列。通常时间复杂度为O(nlogn)，最坏为O(n²)，当数列全相等、已经从小到大排好序或者从大到小排好序后为最坏情况

```python
from typing import List, Optional

NumsList = Optional[List[int]]

class QuickSort:
    def quick_sort(self, nums: NumsList, i: int, j: int) -> NumsList:
        if i > j:
            return nums

        low, high = i, j
        target = nums[i]

        while i < j:
            while i < j and nums[j] >= target:
                j -= 1
            nums[i] = nums[j]
            while i < j and nums[i] <= target:
                i += 1
            nums[j] = nums[i]

        nums[i] = target

        self.quick_sort(nums, low, i-1)
        self.quick_sort(nums, i+1, high)

        return nums

    def main(self, nums: NumsList) -> NumsList:
        return self.quick_sort(nums, 0, len(nums)-1)


if __name__ == "__main__":
    nums = [1, 2, 80, 45, 5, 2, 6, 3]
    print(QuickSort().main(nums))

```

# 归并排序

```python
from typing import List, Optional

NumsList = Optional[List[int]]


class MergeSort:

    def split(self, nums: NumsList) -> NumsList:
        if len(nums) <= 1:
            return nums

        mid = len(nums) // 2
        left = self.split(nums[:mid])
        right = self.split(nums[mid:])

        return self.merge_sort(left, right)
    
    @staticmethod
    def merge_sort(left: NumsList, right: NumsList) -> NumsList:
        i, j = 0, 0
        res = []

        while i < len(left) and j < len(right):
            if left[i] < right[j]:
                res.append(left[i])
                i += 1
            else:
                res.append(right[j])
                j += 1

        res += left[i:]
        res += right[j:]

        return res


if __name__ == "__main__":
    nums = [1, 2, 80, 45, 5, 2, 6, 3]
    print(MergeSort().split(nums))
```