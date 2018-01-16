---
title:
  virtualenv各python版本的虚拟环境创建及常用命令说明
tags:
  - python
  - virtualenv
categories: python学习心得&备忘
abbrlink: 824ac3d0
date: 2017-06-05 10:19:10
---

## win系统
首先
```
pip install virtualenvwrapper-win
```
创建py2环境
```
mkvirtualenv -p C:\python27\python.exe(解释器绝对路径) py2env(虚拟环境名)
```
创建py3环境
```
mkvirtualenv -p C:\python34\python.exe(解释器绝对路径) py3env(虚拟环境名)
```

## linux系统
首先也是一样安装virtualenv
```
pip install virtualenvwrapper
```
然后确定系统中Python的安装位置，比如我的位置在`/usr/bin/`下
创建py2环境
```
virtualenv -p /usr/bin/python2 py2env
```
创建py3环境
```
virtualenv -p /usr/bin/python3 py3env
```
