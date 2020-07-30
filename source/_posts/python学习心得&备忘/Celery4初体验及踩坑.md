---
title: Celery 4 初体验及踩坑
tags:
  - Python
  - celery
  - 消息队列
categories: python学习心得&备忘
abbrlink: d227d93a
date: 2020-07-28 11:37:00
---

# 序

Celery是基于分布式消息传递的开源异步任务队列或作业队列。虽然它支持调度，但其重点是实时操作。现在4版本已经步入稳定，而国内互联网的几乎都是3版本的教程。所以这里记录下4版本下的踩坑及外文解决方案的翻译记录。

# win环境运行celery 4 worker

> Celery 是一个资金最少的项目，因此我们不支持 Microsoft Windows。请不要提出与该平台相关的任何问题。

官方在4版本移除了win平台支持，但是经过查阅，只要使用将并发模式`-P`改为`gevent`或者`eventlet`即可正常启动，**但并不知道会有什么影响，毕竟官方已经不提供支持了**，该启动方法仅适用于本地调试。

附上worker启动脚本

```bash
# celery_worker_start.bat

@echo off

chcp 65001

CLS

echo 正在启动 python 虚拟环境

CALL venv\Scripts\activate.bat

echo 正在启动 celery

celery -A multi_analysis_tasks.celery_app worker -P gevent -l info
```

# flask + celery 启动蓝图循环引用问题

## 项目结构

```python
def register_plugin(application):
    from app.models.base import db
    db.init_app(application)
    with application.app_context():
        db.create_all()


def create_app():
    application = Flask(__name__)
    CORS(application, supports_credentials=True)  # 设置允许跨域
    application.config.from_object('app.config.setting')
    application.config.from_object("app.config.secure")
    register_blueprint(application)
    register_plugin(application)
    return application

app = create_app()
celery_app = make_celery(app)

if __name__ == "__main__":

    app.run(host="0.0.0.0", port=5000, debug=False)

>>> ImportError: cannot import name 'create_blueprint_v1'
```

## 解决方案

celery worker 入口文件和 flask 启动的入口文件分开，worker 的入口文件不需要初始化蓝图，即可解决循环引用的问题。

# 在 celery work 中加入 flask 上下文

> 注意: celery worker 运行的必须是已经推入`flask context`的 celery 对象，后续推入的`context`是无效的。

```python
from celery import Celery
from app.config.setting import CELERY_TASKS_INCLUDE


def make_celery(app):
    celery = Celery(__name__, include=CELERY_TASKS_INCLUDE)
    celery.config_from_object('app.config.celery_conf')

    class ContextTask(celery.Task):
        def __call__(self, *args, **kwargs):
            with app.app_context():
                return self.run(*args, **kwargs)

    celery.Task = ContextTask
    return celery
```

# 成员函数实现 celery task 异步调用

由于支持方法太多bug且没有人修，celery官方在4版本，移除了celery 3 中的`celery.contrib.methods`方法
> I wrote celery.contrib.task_methods as an experiment, but turns out there were some serious bugs
there, for example task retries wouldn't work at all, and since nobody were stepping up to fix it I removed it.

所以尽量将需要异步调用的方法移动到类以外，如果实在是没有办法移动，可以将`celery.contrib.methods`放到项目内，然后实现调用。

```python
from celery import current_app

__all__ = ['task_method', 'task']


class task_method(object):

    def __init__(self, task, *args, **kwargs):
        self.task = task

    def __get__(self, obj, type=None):
        if obj is None:
            return self.task
        task = self.task.__class__()
        task.__self__ = obj
        return task


def task(*args, **kwargs):
    return current_app.task(*args, **dict(kwargs, filter=task_method))

```

使用方法：

```python
from celery import current_app
from celery.contrib.methods import task_method

class A:
@current_app.task(filter=task_method, name='A.foo')
def foo(self, bar):
    ...
```


# 参考
- [flask 基于 Celery 的后台任务](http://docs.jinkan.org/docs/flask/patterns/celery.html)
- [Flask, blueprints uses celery task and got cycle import](https://stackoverflow.com/questions/29670961/flask-blueprints-uses-celery-task-and-got-cycle-import)
- [celery 中文手册](https://www.celerycn.io/ru-men/celery-jian-jie)
- [Hack: 2 Ways to make Celery 4 run on Windows](https://www.distributedpython.com/2018/08/21/celery-4-windows/)
- [using class methods as celery tasks](https://stackoverflow.com/questions/9250317/using-class-methods-as-celery-tasks)
- [where is celery.contrib.methods.task_method of 4.0](https://github.com/celery/celery/issues/3558)