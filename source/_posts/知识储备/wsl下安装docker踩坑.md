---
title: WSL 下安装 docker 踩坑
tags:
  - docker
  - wsl
  - rabbitMQ
categories: 知识储备
abbrlink: 24f059ac
date: 2019-12-11 17:18:00
---

# 起因

项目改微服务化了，于是开始研究 `rabbitMQ` 这个消息队列框架。然后官方推荐使用docker启动，索性在WSL下装 docker 了（太懒了，不想双系统or虚拟机）

# 安装

**-\*-FBI WARNING-\*-**
安装前请注意查阅自己的WSL是否为2.0版，不然请直接看[踩坑 & 解决](http://www.gvoidy.cn/posts/24f059ac/#%E8%B8%A9%E5%9D%91-amp-%E8%A7%A3%E5%86%B3)问题三

因为我的WSL下安装的是Ubuntu，参见[docker官方安装文档](https://docs.docker.com/install/linux/docker-ce/ubuntu/)。

```bash
# 添加源
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -

# 测试指纹是否存在
$ sudo apt-key fingerprint 0EBFCD88

pub   rsa4096 2017-02-22 [SCEA]
      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]

# 设置使用稳定版仓库
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) \
  stable"

# 更新源
$ sudo apt-get update

# 安装
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
```

安装好后先别着急跑`sudo docker run hello-world`, WSL下有很多很多坑。

# 踩坑 & 解决

1. 一定要以 **管理员权限** 打开 WSL 

2. `Cannot connect to the Docker daemon at unix:///var/run/docker.sock` 错误
	需要给`docker.sock` 权限

	```bash
	sudo chmod -R 777 /var/run/docker.sock
	```
	就能成功启动 docker 了
	```bash
	$ sudo service docker restart
	$ sudo service docker status  
	* Docker is running
	```

3. 问题三
	```bash
	$ sudo docker run hello-world

	docker: Error response from daemon: OCI runtime create failed: container_linux.go:346: starting container process caused "process_linux.go:319: getting the final child's pid from pipe caused \"EOF\"": unknown.
	ERRO[0012] error waiting for container: context canceled
	```

	经过多方查阅，发现我的Win10版本是`18362`，而安装WSL的最低版本为`18917`。 WSL 1.0版没有集成完整版Linux内核，所以给出的解决方法是安装`18.X`版的docker

	这里我们先删除docker

	```bash
	# 删除刚刚安装的docker
	$ sudo apt-get remove docker docker-engine docker.io containerd runc

	# 然后查看一下当前可用的所有版本docker
	$ apt-cache madison docker-ce

	docker-ce | 5:19.03.5~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.4~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.3~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.2~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.1~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:19.03.0~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:18.09.9~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:18.09.8~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:18.09.7~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:18.09.6~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:18.09.5~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:18.09.4~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:18.09.3~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:18.09.2~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:18.09.1~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 5:18.09.0~3-0~ubuntu-bionic | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 18.06.3~ce~3-0~ubuntu | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 18.06.2~ce~3-0~ubuntu | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 18.06.1~ce~3-0~ubuntu | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 18.06.0~ce~3-0~ubuntu | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages
 docker-ce | 18.03.1~ce~3-0~ubuntu | https://download.docker.com/linux/ubuntu bionic/stable amd64 Packages

 	#最后选择一个旧版本安装, 我选的最后一个
	$ sudo apt-get install docker-ce=18.03.1~ce~3-0~ubuntu
	```

	然后就能启动了~

	```bash
	$ sudo docker run hello-world

	Hello from Docker!
	This message shows that your installation appears to be working correctly.

	To generate this message, Docker took the following steps:
	 1. The Docker client contacted the Docker daemon.
	 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
	    (amd64)
	 3. The Docker daemon created a new container from that image which runs the
	    executable that produces the output you are currently reading.
	 4. The Docker daemon streamed that output to the Docker client, which sent it
	    to your terminal.

	To try something more ambitious, you can run an Ubuntu container with:
	 $ docker run -it ubuntu bash

	Share images, automate workflows, and more with a free Docker ID:
	 https://hub.docker.com/

	For more examples and ideas, visit:
	 https://docs.docker.com/get-started/
	```