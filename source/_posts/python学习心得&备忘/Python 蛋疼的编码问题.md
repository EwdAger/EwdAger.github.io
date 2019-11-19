---
title: Python 蛋疼的编码问题
tags:
  - Python2
  - Python
  - 编码
categories: python学习心得&备忘
abbrlink: 7ed6f334
date: 2018-05-02 12:44:00
---

Python 的编码问题早就困扰我太久了， 但一直没有看到比较通俗易懂的专门介绍 Python 编码问题的文章。 正好今天刷知乎看到了非常不错的文章， 这里稍微抛砖引玉归纳下。

## Unicode 和 UTF-8

知识储备：

- ASCII 占1个字节，只支持英文
- GBK GB2312的升级版，支持21000+汉字
- Shift-JIS 日本字符
- ks_c_5601-1987 韩国编码
- TIS-620 泰国编码

由于每个国家都有自己的字符，所以其对应关系也涵盖了自己国家的字符，但是以上编码都存在局限性，即：仅涵盖本国字符，无其他国家字符的对应关系。 应运而生出现了万国码(Unicode)，他涵盖了全球所有的文字和二进制的对应关系。

Unicode解决了字符和二进制的对应关系，但是使用unicode表示一个字符，太浪费空间。例如：利用unicode表示“Python”需要12个字节才能表示，比原来ASCII表示增加了1倍。

由于计算机的内存比较大，并且字符串在内容中表示时也不会特别大，所以内容可以使用unicode来处理，但是存储和网络传输时一般数据都会非常多，那么增加1倍将是无法容忍的！！！

为了解决存储和网络传输的问题，出现了Unicode Transformation Format，学术名UTF，**即：对unicode中的进行转换，以便于在存储和网络传输时可以节省空间!**



- UTF-8： 使用1、2、3、4个字节表示所有字符；优先使用1个字符、无法满足则使增加一个字节，最多4个字节。英文占1个字节、欧洲语系占2个、东亚占3个，其它及特殊字符占4个
- UTF-16： 使用2、4个字节表示所有字符；优先使用2个字节，否则使用4个字节表示。
- UTF-32： 使用4个字节表示所有字符；

总结： **UTF 是为unicode编码设计的一种在存储和传输时节省空间的编码方案。**

## 编码的转换

对于历史遗留的以 GBK 编码编写的程序，我们可以不将其重新编码成 UTF-8 让其在没有安装 GBK 编码的终端上不乱码。 因为 Unicode 包含它与所有国家编码的映射关系， 所有只需要把数据从硬盘读到内存里， 转成 Unicode 来显示就可以了。
由于所有的系统、编程语言都默认支持unicode，那你的gbk软件放到美国电脑 上，加载到内存里，变成了unicode,中文就可以正常展示啦。

## Python3 的执行过程

在看实际代码的例子前，我们来聊聊，python3 执行代码的过程

1. 解释器找到代码文件，把代码字符串按文件头定义的编码加载到内存，转成unicode
2. 把代码字符串按照语法规则进行解释，
3. 所有的变量字符都会以unicode编码声明


## 编码转换过程

在 py2 和 py3 下分别运行下面这段程序

```python
# coding: utf-8
s = '你好'
print(s)
```

Python3:
```python
'你好'
```

Python2:
```python
'浣犲ソ'
```

好了，这里就是最恶心的 Python2 的编码问题了。

这里使用的是 Windows cmd 默认的 GBK 编码运行的程序。 为什么py3正常，py2就显示二进制字节了呢。

因为到了内存里 python3 解释器把 utf-8 转成了 Unicode，而 python2 的默认编码是 ASCII ，py2 解释器仅以文件头声明的编码去解释这段代码， 加载到内存后，并不会主动转成 Unicode ，也就是说你的文件编码是以 utf-8 的信使加载到内存的， 所以是乱码。

在 Windows 上，字符串只有**以GBK格式显示**或者编码是**Unicode**才能正常显示。

这种情况，只能自己手动`decode`成 Unicode 了。

> UTF-8 --> decode 解码 --> Unicode
> Unicode --> encode 编码 --> GBK / UTF-8 ..

```python
# coding: utf-8
s = '你好'

s2 = s.decode('utf-8')
print(s2)                   # 你好
print(typr(s2))             # <type 'unicode'>

s3 = s2.encode("GBK")
print(s3)                   # 你好
print(type(s3))             # <type 'str'>
```

![转换规则](https://gvoidy-1251878576.cos.ap-chengdu.myqcloud.com/1.png)

### 如何验证编码转对了呢？

1. 查看数据类型， python2里有专门的 Unicode 类型
2. 查看 Unicode 编码映射表

unicode字符是有专门的unicode类型来判断的，而 utf-8, gbk 编码的字符都是`str`。

## python bytes 类型

Python2：
```python
>>> s = "你好"
>>> print s
你好
>>> s
'\xc4\xe3\xba\xc3'
>>> s.decode('GBK')    # CMD 编码格式为 GBK
u'\u4f60\u597d'        # 在 Unicode 编码表中对应的位置
```

首先， python2 是以 bytes 形式存储非英文字符串，所以`bytes`类型就是`str`

```python
>>> s = '你好'
>>> type(s)
<type 'str'>
```

### Python3 的变革

Python3 中终于把字符串的编码从 ASCII 改为了 Unicode ，并且把`str`和`bytes`做了明确的区分，`str`就是 Unicode 格式的字符， `bytes`就是单纯的二进制。

但是把 Unicode 编码成 GBK 后，字符串变成了`bytes`格式。 可能这是为了告诉你想在py3里看字符，必须得是unicode编码，其它编码一律按bytes格式展示。

```python
>>> s = '你好'
>>> s
'你好'
>>> s2 = s.encode('GBK')
>>> s2
b'\xc4\xe3\xba\xc3'
>>> type(s2)
<class 'bytes'>
```

原文出自[文章链接](https://www.zhihu.com/question/31833164/answer/381137073)作者知乎用户：Alex-金角大王

这位大神写的太好了，这篇归纳都差不多算是照着原文重打一遍了。。。
