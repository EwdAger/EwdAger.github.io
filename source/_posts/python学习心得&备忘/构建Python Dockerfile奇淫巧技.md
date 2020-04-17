---
title: 构建Python Dockerfile的奇淫巧技
tags:
  - python
  - docker
categories: python学习心得&备忘
abbrlink: ed4c24bb
date: 2020-04-16 16:35:10
---

# 镜像构建

## 前言

最简单的情况下，如果我们使用官方python镜像，构建我们的容器会无敌庞大。因为他帮我们预置了许许多多类库。同时我们直接使用`RUN pip install /xxx/requirements.txt`安装环境时，每次构建镜像都会从pip仓库里面拉包，也会非常慢。

```Dockerfile
FROM python:3.7
COPY . /app
WORKDIR /app
RUN pip install -r requirements.txt
CMD ["uwsgi", "--ini", "/xxx/uwsgi.ini"]
```

## 构建requirements缓存（使用空间换时间）

```Dockerfile
FROM python:3.7
COPY requirements.txt /
RUN pip install -r /requirements.txt
COPY src/ /app
WORKDIR /app
CMD ["uwsgi", "--ini", "/xxx/uwsgi.ini"]
```

用这种方式重写我们的 Dockerfile，可以利用 Docker 的层缓存，如果 requirements.txt 文件不变，则跳过安装 pip 包。

## 使用alpine镜像（使用时间换空间）

```Dockerfile
FROM python:3.6-alpine

COPY requirements.txt /

RUN pip install --upgrade pip -i https://pypi.douban.com/simple \
    && pip install -r /requirements.txt -i https://pypi.douban.com/simple

RUN mkdir /mailAlarm

WORKDIR /mailAlarm

COPY . /mailAlarm

VOLUME /mailAlarm/config

RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories \
    && apk add --no-cache tzdata \
    && ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo 'Asia/Shanghai' >/etc/timezone

CMD ["python", "-u", "/mailAlarm/run.py"]
```

这种方式构建镜像就能得到很小的镜像，但是需要额外安装部分pip包所依赖的类库。因为alpine版本镜像默认是只安装python环境所需要的基础类库。

> 特别注意，某些工具类的包编译安装完pip包后可以使用`apk del `删除
> ※ 常用包： .build-deps gcc musl-dev

# 其他神奇的方法

## 链接log文件到docker日志流

```Dockerfile
ln -sfT /dev/stdout "/mfa/servers.log"
```

## 设置时区
> 如果使用的是`alpine`镜像，需要安装`tzdata`，且不能删除

```Dockerfile
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime \
    && echo 'Asia/Shanghai' >/etc/timezone
```
