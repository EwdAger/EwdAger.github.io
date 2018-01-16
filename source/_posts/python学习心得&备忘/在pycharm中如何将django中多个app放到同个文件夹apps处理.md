---
title: 在pycharm中如何将django中多个app放到同个文件夹apps处理
tags:
  - Django
  - Web框架
categories: python学习心得&备忘
abbrlink: ab3d95d5
date: 2017-06-05 11:20:10
---

新建apps文件夹后mark为source目录，然后在Setting中import方式为

```
from message import views
```

但这样run manage.py task时会报模块不存在的错误

```
ImportError: No module named message
```

此时要在Setting中设置app的路径

```
sys.path.insert(0,os.path.joinBASE_DIR,'apps'))
```
注意：此时的
`
from message import views
`
必须写在设置路径语句之后
还有记得要注册app哦:)
