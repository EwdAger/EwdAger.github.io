---
title: linux环境下使用mono运行asf挂卡
date: 2017-06-13 20:54:10
tags: 
 - Steam
 - Linux
categories: Steam骚操作
---

杰瑞包大好评啊，但是带来的后果就是挂卡挂不完了。然而手里的服务器全是linux环境的并不支持C#写的ASF，所以用mono f**k之。
<!-- more -->
##Ubuntu篇
接下来介绍如何安装mono，以Ubuntu 14.04为例。

**1. 运行下面代码授权注册repo源并更新软件列表：**
```
$ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF  
$ echo "deb http://download.mono-project.com/repo/debian wheezy main" | sudo tee /etc/apt/sources.list.d/mono-xamarin.list  
$ sudo apt-get update  
```

**2. 安装mono**
```
$ sudo apt-get install mono-complete  
```
**3. 测试mono是否安装成功**
```
mono -V  //如果没有提示错误就可以啦
```
**3. 运行asf**
首先通过ftp工具将配置好的asf传到服务器上。然后新建一个窗口``` screen -S ASF```，最后进入asf的目录再运行asf就行啦~
```
cd /opt/ASF
mono ASF.exe
```
##[CentOS篇](https://steamcn.com/t169180-1-1)
这里转一个教程，当做留档了。
**1. 添加yum源**
```
rpm --import "http://keyserver.ubuntu.com/pks/lookup?op=get&search=0x3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF"    
yum-config-manager --add-repo http://download.mono-project.com/repo/centos/
```
**2. 有可能提示找不到yum-config-manager ,这个是因为系统默认没有安装这个命令,这个命令在yum-utils 包里,可以通过命令```yum -y install yum-utils```安装。**

**3. 安装mono**
```
yum -y install mono-complete
```
**4. 以下参照上面的3步以后**

PS：关于“最小化Screen”，因为开启了screen后就不能进行其他操作了，我们的服务器当然不只是为了挂卡而存在的，所以可以通过按住```Ctrl+A+D```“最小化”screen窗口。

PPS：还有就是恢复的话，在终端里输入```screen -r ASF```就可以了

PPPS：如果不想挂卡了，就输入以下命令杀掉进程。
```
screen -ls    #显示所有的screen窗口名字和进程号
kill [进程号]
```
附上成功图：

![ASF](http://upload-images.jianshu.io/upload_images/5433252-df15fd3b1a6cd0a2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
