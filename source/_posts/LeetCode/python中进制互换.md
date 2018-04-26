---
title: python中进制互换
tags:
  - LeetCode
  - python
categories: LeetCode刷题总结
abbrlink: 8924f3f8
date: 2018-04-26 19:25:00
---

## 十进制转二进制

bin() 返回一个整数 int 或者长整数 long int 的二进制表示。

```python
>>>bin(10)
'0b1010'
>>> bin(20)
'0b10100'
```

## 二进制转十进制

```python
>>>int('0b1010', 2)
'10'
>>>int('0b10100', 2)
'20'
```

## LeetCode 相关题目 [476. 数字的补数](https://leetcode-cn.com/problems/number-complement/description/)
<!-- more -->

给定一个正整数，输出它的补数。补数是对该数的二进制表示取反。

**注意:**

1. 给定的整数保证在32位带符号整数的范围内。
2. 你可以假定二进制数不包含前导零位。

**示例1：**
> 输入: 5
> 输出: 2
> 解释: 5的二进制表示为101（没有前导零位），其补数为010。所以你需要输出2。

**示例2：**
> 输入: 1
> 输出: 0
> 解释: 1的二进制表示为1（没有前导零位），其补数为0。所以你需要输出0。

### 我的解法（83.7%）：

暴力解法，把数字转换成二进制字符串去掉`0b`， 用遍历的方法取字符串补码， 再转回十进制数。

```python
class Solution:
    def findComplement(self, num):
        """
        :type num: int
        :rtype: int
        """
        str_num = str(bin(num))[2:]
        anti_num = []
        for e, i in enumerate(str_num):
            if i == '0':
                anti_num.append("1")
            else:
                anti_num.append("0")
        anti_num.insert(0, "0b")
        anti_num = "".join(anti_num)
        return int(anti_num, 2)       
```

### 最优解：

```python
class Solution:
    def findComplement(self, num): 
        """
        :type num: int
        :rtype: int
        """
        n = len(bin(num))-2
        return num^(2**n-1)
```

思路是，取出去除`0b`的二进制数‘长度’ `n`, 通过`n`求出`num`‘位数’全置`1`的十进制数， 在与`num`亦或即可得到补数。（补数的性质啊喂。。。这都没想出来）