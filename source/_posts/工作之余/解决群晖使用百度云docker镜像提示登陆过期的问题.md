---
title: 解决群晖使用百度云docker镜像提示登陆过期的问题
tags:
  - 群晖
categories: 工作之余
abbrlink: 304b0279
date: 2020-10-11 20:54:10
---

# 0x00

群晖中使用的百度套件是基于`VNC + Linux版百度云`打包的docker镜像实现的，但是作者因为稳定性原因通过修改检测更新的hosts方法将百度云版本锁定在3.0.1.2版本。可是这在部分机器上不生效，那么我们反之道而行，自己更新容器里的百度云版本。

# 解决方案

1. 去百度云官网下载`.deb`版百度云: [下载地址](https://pan.baidu.com/download)

2. 将下好的镜像放到群晖已挂载的`baidudownload`文件夹里

3. 进入容器中的`baidudownload`目录

4. 执行 `sudo dpkg -i baidunetdisk_3.4.1_amd64.deb`

5. 将hosts文件中`127.0.0.1 update.pan.baidu.com`这一行注释掉

然后群晖中的百度云套件就不报登陆过期的问题了