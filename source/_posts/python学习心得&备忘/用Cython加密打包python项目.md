---
title: 用Cython加密打包python项目
tags:
  - cython
  - 打包
categories: python学习心得&备忘
abbrlink: a2a2ab6a
date: 2019-09-02 11:26:10
---

# 使用

将下述代码保存为`setup.py`至需打包项目根目录，安装`cython`后执行`python setup.py`即可打包。



```python
import sys, os, shutil, time
from distutils.core import setup
from Cython.Build import cythonize

start_time = time.time()
curr_dir = os.path.abspath('.')
parent_path = sys.argv[1] if len(sys.argv) > 1 else ""
setup_file = __file__.replace('/', '\\')
build_dir = "build"
build_tmp_dir = build_dir + "/temp"

s = "# cython: language_level=3"


def get_py(base_path=os.path.abspath('.'), parent_path='', name = '', excepts=(), copyOther=False, delC = False):
    """
    获取py文件的路径
    :param base_path: 根路径
    :param parent_path: 父路径
    :param excepts: 排除文件
    :return: py文件的迭代器
    """
    full_path = os.path.join(base_path, parent_path, name)
    for filename in os.listdir(full_path):
        full_filename = os.path.join(full_path, filename)
        if os.path.isdir(full_filename) and filename != build_dir and not filename.startswith('.'):
            for f in get_py(base_path, os.path.join(parent_path, name), filename, excepts, copyOther, delC):
                yield f
        elif os.path.isfile(full_filename):
            ext = os.path.splitext(filename)[1]
            if ext == ".c":
                if delC and os.stat(full_filename).st_mtime > start_time:
                    os.remove(full_filename)
            elif full_filename not in excepts and os.path.splitext(filename)[1] not in ('.pyc', '.pyx'):
                if os.path.splitext(filename)[1] in ('.py', '.pyx') and not filename.startswith('__'):
                    path = os.path.join(parent_path, name, filename)
                    yield path
        else:
            pass


def pack_pyd():
    # 获取py列表
    module_list = list(get_py(base_path=curr_dir, parent_path=parent_path, excepts=(setup_file,)))
    try:
        setup(
            ext_modules=cythonize(module_list, compiler_directives={'language_level': "3"}),
            script_args=["build_ext", "-b", build_dir, "-t", build_tmp_dir],
        )
    except Exception as ex:
        print("error! ", str(ex))
    else:
        module_list = list(get_py(base_path=curr_dir, parent_path=parent_path, excepts=(setup_file,), copyOther=True))

    module_list = list(get_py(base_path=curr_dir, parent_path=parent_path, excepts=(setup_file,), delC=True))
    if os.path.exists(build_tmp_dir):
        shutil.rmtree(build_tmp_dir)

    print("complate! time:", time.time() - start_time, 's')


def delete_c(path='.', excepts=(setup_file,)):
    '''
    删除编译过程中生成的.c文件
    :param path:
    :param excepts:
    :return:
    '''
    dirs = os.listdir(path)
    for dir in dirs:
        new_dir = os.path.join(path, dir)
        if os.path.isfile(new_dir):
            ext = os.path.splitext(new_dir)[1]
            if ext == '.c':
                os.remove(new_dir)
        elif os.path.isdir(new_dir):
            delete_c(new_dir)


if __name__ == '__main__':
    try:
        pack_pyd()
    except Exception as e:
        print(str(e))
    finally:
        delete_c()

```

# 常见问题

1. 出现`Unable to find vcvarsall.bat`错误

    需安装对应版本`VC++` [下载地址](http://go.microsoft.com/fwlink/?LinkId=691126)

2. 其他文件都能打包，某一文件迷之无法打包
    检查文件名是否包含非法字符（比如`-`）

3. Linux下打包失败或卡住不动
    检查是否安装依赖`yum install python-devel gcc`,如果确定安装依赖，可以稍微等一等，Linux内存不足编译稍大型文件会要很长时间

4. 打包后出现无法调用某模块
    检查对应文件是否循环`import`(如A文件首行调用B，B首行调用A，默认解释器执行不会出错，编译后会出错)

5. 打包后路径出现问题
    在每个需要打包的文件夹中加入空的`__init__.py`文件用于判断路径，如果有非空的`__init__.py`文件，记得打包后复制进对应文件夹

# 尚未解决的问题

1. `__init__.py`判断路径打包和`import`相对路径绝对路径复用，某些情况下打包后会出现无法定位某些模块的奇怪问题