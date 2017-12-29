---
title: git的ssh key创建
date: 2017-08-14 16:56:10
tags: 
 - Git
categories: 不知道怎么归类
---

Git是分布式的代码管理工具，远程的代码管理是基于ssh的，所以要使用远程的git则需要ssh的配置。如果未配置ssh key将无法clone远程代码仓库到本地。
- 第一步.创建user.name和email
```
git config --global user.name "EwdAger"
git config --global user.email "ewdager@hotmail.com"
```
<!-- more -->
- 第二步.生成SSH密钥：
1. 查看是否已经有了ssh密钥：```cd ~/.ssh```,Windows用户的路径在```C:\Users\EwdAger\.ssh```下
如果是刚安装git则不会有此文件夹，有则备份删除
2. 生成ssh key
```sudo ssh-keygen -t rsa -C "ewdager@hotmail.com"```
然后按三下回车
最后得到了两个文件：id_rsa和id_rsa.pub
3. 在github上添加ssh密钥，这要添加的是“id_rsa.pub”里面的公钥
用编辑器打开id_rsa.pub复制全部内容到github的settings的SSH and GPG keys设置中，title取个好记的名字就行。
![](http://upload-images.jianshu.io/upload_images/5433252-0697e396457b08da.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
4. 然后就能使用clone了
