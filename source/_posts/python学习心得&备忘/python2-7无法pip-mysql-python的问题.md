---
title: python2.7无法pip mysql-python的问题
tags:
  - python2
categories: python学习心得&备忘
abbrlink: cb0b0c45
date: 2017-06-05 11:11:10
---

出现问题：
```
> pip install MySQL-python

_mysql.c(42) : fatal error C1083: Cannot open include file: 'config-win.h': No such file or directory error: command '"C:\Users\fnngj\AppData\Local\Programs\Common\Microsoft\Visual C ++ for Python\9.0\VC\Bin\amd64\cl.exe"' failed with exit status 2

```

解决方法：
在http://www.lfd.uci.edu/~gohlke/pythonlibs/#mysql-python下载对应的包版本。
然后在命令行执行

```pip install MySQL_python-1.2.5-cp27-none-win_amd64.whl```
