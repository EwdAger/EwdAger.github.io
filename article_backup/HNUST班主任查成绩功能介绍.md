---
title: HNUST班主任查成绩功能介绍
tags:
  - Django
  - Python
categories: python学习心得&备忘
abbrlink: 456ed2f0
date: 2018-02-27 19:10:00
---

# 概述

由于湖科大教务网中没有班主任这个属性，所以各班班主任查询同学们的成绩缺乏一定的途径而诞生了这个项目。

本项目实现主要逻辑是首先使用模拟登陆至教务网，将各年级各专业的成绩数据页面下载到本地，再通过爬虫将相关数据存入数据库。前端部分再通过后端部分编写的各个接口访问查询到相关成绩数据。



## 开发环境

本程序基于Python27 Django 1.11.2开发，使用的相关第三方模块请参考[requirements.txt](#)。

## 项目结构

本项目采用Django的[MTV结构](https://docs.djangoproject.com/en/1.11/faq/general/#django-appears-to-be-a-mvc-framework-but-you-call-the-controller-the-view-and-the-view-the-template-how-come-you-don-t-use-the-standard-names)构成，所以分爬虫、路由、Models、Templates、Views五部分介绍。

<!-- more -->
# 爬虫部分jwc_spider.py

## login(self)

- 辅导员分别账号密码对应字典data中的USERNAME、PASSWORD两个值中，验证码会通过[captcha_verify](https://github.com/EwdAger/Captcha_Verify)文件自动识别。

- zy列表中的十六进制字符串对应查询专业成绩post数据中字段zy的值，如需添加请自行使用抓包工具抓取。

- 运行该函数会按zy列表填写的专业年级顺序下载成绩页面至`./ScoreQuery/page/Score/`下。

- **tips: 如需替换辅导员账号，请先登录教务网，将信息中心中的自定义分页数量选项调整至1000，不然会因为成绩页面需要分页而导致部分同学信息缺失**

## Score_keys_spdier(self, num)

- 获取当前成绩页面所有的课程名

## Score_soup(self, num, soup=None)

- 初始化时定位当前成绩页面下第一个学生数据的html节点，并返回该节点。

- 初始化后跳转至下一学生信息的html节点并返回。

## Score_spider(self, num, value_soup=None)

- 从Score_soup()获取的html节点中取出名字，学号，成绩等信息放入list并返回

## Student_spdier(self, grade)

- 学生学籍信息爬虫，用于使用学号或姓名查询学生所在班级。(之前下载的成绩页面没有学生-班级的对应关系)

- 该部分使用手工下载学生学籍信息页面，每有新生入校必须将新生学籍信息页面放入`./ScoreQuery/page/Student`下

- 当前页面下所有学生学号、学生姓名和所在班级分别存入对应list并返回。

# Models

# 路由部分urls.py

To be Continue...