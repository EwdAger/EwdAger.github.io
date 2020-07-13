---
title: uwsgi 多进程导致数据库连接丢失的踩坑记录
tags:
  - uwsgi
  - Flask-SQLAlchemy
  - 多进程
  - Python
  - Flask
categories: python学习心得&备忘
abbrlink: cca84fb6
date: 2020-04-28 11:35:10
---

# 起因

项目使用的 Flask+SQLAlchemy+uwsgi ，突然有一天编写了一个有对数据库高并发的接口。然后其他本来正常的接口就偶尔会出现`404`错误，且必须重启服务才能解决。

## 试验①

以为是MySQL连接池和超时时间导致的，反复查看发现并没有什么问题。然后怀疑到是不是python对MySQL的连接驱动导致的。

项目里使用的pymysql被公认为是比较慢的连接驱动。索性换成了mysqldb。
![MySQL连接驱动性能对比](https://s1.ax1x.com/2020/04/28/J4yvVJ.png)

结果只是使触发这种bug的频率稍微降低了一点

## 试验②

后来就怀疑到是不是uwsgi起多进程的时候触发了什么奇怪的bug，结果一搜就在Stack Overflow上发现了宝藏。

>When working with multiple processes with a master process, uwsgi initializes the application in the master process and then copies the application over to each worker process. The problem is if you open a database connection when initializing your application, you then have multiple processes sharing the same connection, which causes the error above.

简单翻译一下，就是uwsgi启动多进程时，会启动一个主进程初始化所有的app（其中包括数据库连接），然后将所有app复制到其他进程中。**这！就！导！致！了！**所有进程**全部共用一个MySQL的连接**

如果在`uwsgi.ini`中添加参数`lazy-apps=true`，即可让各个进程都创建自己的app。即所有进程都有属于自己的MySQL连接了。

[详细地址： using-flask-sqlalchemy-in-multiple-uwsgi-processes](https://stackoverflow.com/questions/34252892/using-flask-sqlalchemy-in-multiple-uwsgi-processes)