---
title: Ubuntu下搭建环境+安装wordpress
date: 2017-04-15 16:35:10
tags:
 - Linux
categories: 服务器相关问题&备忘
---

LAMP 是Linux, Apache, MySQL, PHP, perl的缩写. 指在linux上安装Apache2，MySQL, PHP等软件包所建立的网站运行平台，是目前中小网站主要的运行环境。
<!-- more -->
1.1 安装Apache2

`sudo apt-get install apache2`
1.2 安装MySQL5

`sudo apt-get install mysql-server mysql-client`
中途需要设置root密码

1.3 安装PHP5

```
sudo apt-get install php5 libapache2-mod-php5 php5-mysql php5-curl php5-gd php5-intl php-pear php5-imagick php5-imap php5-mcrypt php5-memcache php5-ming php5-ps php5-pspell php5-recode php5-snmp php5-sqlite php5-tidy php5-xmlrpc php5-xsl php5-xcache libssh2-php 
```
1.4 安装phpMyAdmin

```
sudo apt-get install phpmyadmin
...
Web server to reconfigure automatically: < -- apache2
Configure database for phpmyadmin with dbconfig-common? <-- No 
...
```
2. 初始化数据库
```
sudo mysql -u root -p
Enter Password:
...
mysql>  CREATE DATABASE wordpressdb;
mysql>  CREATE USER wordpressuser@localhost IDENTIFIED BY '你的密码';
mysql>  GRANT ALL PRIVILEGES ON wordpressdb.* TO wordpressuser@localhost;
mysql> FLUSH PRIVILEGES;
mysql> exit 
```
重启服务

``` 
sudo service apache2 restart
sudo service mysql restart 
```
3. 下载并配置WordPress
```
$ mkdir temp
$ cd temp
$ wget http://wordpress.org/wordpress-4.x.tar.gz（最新版本号)
$ tar zxf wordpress-4.x.tar.gz -C /var/www/html/
$ mkdir -p /var/www/html/wordpress/wp-content/uploads
```

转自http://www.cnblogs.com/R0b1n/p/5224070.html
http://blog.csdn.net/ansencumt/article/details/9000436
目前配置wp-config.php
