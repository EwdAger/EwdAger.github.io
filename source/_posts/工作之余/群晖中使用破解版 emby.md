---
title: 群晖中使用破解版 emby
tags:
  - 群晖
categories: 工作之余
abbrlink: 38ad339f
date: 2020-10-11 20:54:10
---

# 0x00

本文基本来源 [Yukino's 理想乡](https://neko.re/) 的 [emby 破解版完全食用方法](https://neko.re/archives/128.html)

但是文中没有提供 docker 版的安装方法和相关配置！！所以这里完善一下！

套件版方案使用起来很舒服，但是需要 fan 墙刮削。docker 版可以以一种巧妙的方式刮削，但是配置起来麻烦。

# 套件版 emby 破解

1. 因为使用 emby 套件源，在国内难以访问，[所以我们从官网上下载 emby 并安装](https://emby.media/synology-server.html)

2. 安装好后关闭 emby

3. 使用脚本破解 `wget https://neko.re/wp-content/uploads/simple-file-list/NyaaHost.sh ; sh NyaaHost.sh` (再次鸣谢[Yukino's 理想乡](https://neko.re/))

4. 进入并配置 emby

5. 使用 Chrome 安装 URLRedirector(Chrome, Firefox) 插件，添加用户规则
原始地址 https://mb3admin.com ，目标地址 https://crackemby.neko.re ，然后确认并保存，别忘了勾选重定向。

6. Chrome 访问 emby 地址，emby 高级版秘钥里随便填个字符串点保存就破解成功了\~！


# Docker 版 emby 安装及破解

先附上容器启动命令

```bash
docker create \
–name=emby \
–device=/dev/dri/renderD128:/dev/dri/renderD128 \
–device=/dev/dri/card0:/dev/dri/card0 \
-p 
yukinococo/emby_crack:unix-x64
```