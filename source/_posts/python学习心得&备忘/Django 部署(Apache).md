---
title: Django 部署(Apache)
tags:
  - Django
  - Web框架
categories: python学习心得&备忘
abbrlink: 725a38d2
date: 2018-06-15 17:32:10
---

## 安装
```bash
sudo apt-get install python2.7.12     // 安装对应版本python
sudo apt-get install apache2          // 安装apache
sudo apt-get install libapache2-mod-wsgi  // 安装apache的wsgi组件
sudo apt-get install pip              // 安装python包管理
```
如果项目内有`requrements.txt`文件，进行如下操作安装项目依赖
```
pip install -r requrements.txt
```

## 配置 Apache

首先将django项目放入`/var/www/`目录下，然后修改`/etc/apache2/site-enabled/000-default.conf/`文件

<!--more-->
```bash
<VirtualHost *:80>

...略...

	# 配置python环境的路径，此处已系统环境为例

	WSGIDaemonProcess Score python-path=/var/www/Score:/usr/bin/python/lib/python2.7/site-packages
	WSGIProcessGroup Score
	WSGIScriptAlias / /var/www/Score/Score/wsgi.py

	# 配置django静态文件目录

	Alias /static/ /var/www/Score/ScoreQuery/static/
	<Directory /var/www/Score/ScoreQuery/static/>
		Require all granted
	</Directory>
</VirtualHost>
```

## 收集 Django 静态文件

进入`manage.py`目录输入
```bash
python manage.py collectstatic
```

## 更改项目读写权限

```bash
chown -R www-data:www-data /var/www/Score
```

## 重启 Apache
```bash
service apache2 restart
```
