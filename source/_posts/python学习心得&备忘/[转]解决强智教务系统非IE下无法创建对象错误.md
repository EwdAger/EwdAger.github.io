---
title:
  (转)解决强智教务系统非IE下无法创建对象错误
tags:
  - 教务网
  - 爬虫
categories: python学习心得&备忘
abbrlink: 824ac3d0
date: 2017-06-19 20:18:10
---

** 最近要弄教务网的模拟登陆，但苦于教务网只兼容IE8以下的浏览器，不能用chrome强大的F12抓包就很烦，然后发现Fly俊大佬弄了一个相当强的插件啊。但是怕[Fly俊大佬的博客](http://www.qiujun.me)失效，所以私自留了个档。侵删啊大佬~ **

*** 以下内容均为转载 ***

学校教务系统由于长期缺乏必要的维护，目前依旧只兼容IE8及以下浏览器
![IE8及以下](http://upload-images.jianshu.io/upload_images/5433252-4ac3942a591ffb91.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
使用非IE浏览器访问登陆界面是没有问题，但是登录进去后就会报错，并且所有功能都无法使用
![无法创建对象](http://upload-images.jianshu.io/upload_images/5433252-1eb9c8b6ef279963.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这对于使用非IE浏览器的用户简直就是噩梦，特别对于非Windows用户，以下是简单的解决办法
<!-- more -->
**学长的修复**

一位高能的学长在无法忍受这个问题后，进行了部分修复每次使用非IE浏览器访问后，替换一个JS文件为学长修改后的文件就可以正常使用了但是这个学长修复的结果又产生了一些新问题
无法创建对象问题解决了，但是又产生了新问题，所有的内容都重复了两遍
替换JS文件对小白用户来说是个难点

[](https://qiujun.me/post/resolve-bug-for-qzsoft/#新的解决办法)**新的解决办法**

经过查找、调试，终于是找到了内容重复的原因，内容是不再重复了然而怎么让普通用户也能替换JS文件呢感觉最好的办法还是写个浏览器插件，自动替换JS，不过需要注意以下几个问题。
插件只针对Chrome浏览器编写，不过360浏览器等使用Chrome内核的浏览器可能能够使用，不过未测试。

由于教务网浏览器兼容问题众多，所以此插件并不能完美修复所有问题。

[](https://qiujun.me/post/resolve-bug-for-qzsoft/#插件使用教程)**插件使用教程**

[插件原地址>>
](https://qiujun.me/uploads/resolve-bug-for-qzsoft/%E6%B9%96%E5%8D%97%E7%A7%91%E6%8A%80%E5%A4%A7%E5%AD%A6%E6%95%99%E5%8A%A1%E5%A4%84%E4%BF%AE%E5%A4%8D%E6%8F%92%E4%BB%B6.crx)
[度盘密码udn8>>](http://pan.baidu.com/s/1kVFortH)

在谷歌浏览器中打开 [chrome://extensions/
](chrome://extensions/)

将下载好的插件拖放
至上一步骤打开的 插件管理页面，然后会弹出如下窗口，点击添加即可
![添加插件](http://upload-images.jianshu.io/upload_images/5433252-0b8cb0fa6c00b1e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后就可以开开心心的打开教务网了
