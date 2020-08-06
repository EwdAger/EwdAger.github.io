---
title: Rancher 高可用部署
tags:
  - Linux
  - docker
  - rancher2
  - k8s
  - DevOps
categories: 云原生学习笔记
abbrlink: 77c36b93
date: 2020-08-06 10:46:10
---

# 0x00

> 其实官网已经有了无坑且完备的高可用部署方案 [官方部署方案链接](https://rancher2.docs.rancher.cn/docs/installation/k8s-install/helm-rancher/_index)，但是太过翔实，这里只是记录一下自己的部署方案 

# 说明

本教程是基于k3s安装Rancher Server，从Rancher V2.4开始支持在K3s集群安装，K3s比RKE更新，易于使用且更轻量，全部组件都打包在了一个二进制文件里。

# 前置条件

## mysql已安装，配置账户及访问权限

创建可读写rancher database的账户，限定可访问ip为rancher server所在服务器ip

```bash
create user rancher identified by 'rancher#1Yer';
grant all privileges on rancher.* to rancher@'<yourNodeIP1>' identified by 'rancher#1Yer';
grant all privileges on rancher.* to rancher@'<yourNodeIP2>' identified by 'rancher#1Yer';
flush privileges;
```

## 安装docker

```bash

# 安装依赖yum install -y yum-utils device-mapper-persistent-data lvm2
# 添加源
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
 
# 更新yum缓存
yum clean all
yum makecache fastyum -y
 
# 安装docker ce
yum install docker-ce-19.03.5-3.el7.x86_64
 
# 通过systemctl启动服务
systemctl start docker
  
# 开机自启动
systemctl enable docker
 
# 关闭docker 将docker存储位置转为/data目录（防止系统盘被占满),data目录根据实际情况修改
systemctl stop docker
 
mv /var/lib/docker /data/docker
ln -s /data/docker /var/lib/docker

```

## 安装kubectl

```bash
wget https://storage.googleapis.com/kubernetes-release/release/v1.18.2/bin/linux/amd64/kubectl
 
chmod +x ./kubectl
 
sudo ln -s ./kubectl /usr/local/bin/kubectl
 
kubectl version --client
```

## 安装Helm

```bash

# 下载安装helm
wget https://get.helm.sh/helm-v3.2.0-linux-amd64.tar.gz
tar -zxvf helm-v3.2.0-linux-amd64.tar.gz
mv linux-amd64/helm /usr/local/bin/helm
 
# 添加 Helm Chart 仓库
helm repo add rancher-stable> https://releases.rancher.com/server-charts/stable

```

# 部署 k3s 集群

在待部署的机器上分别执行

```bash
curl -sfL https://docs.rancher.cn/k3s/k3s-install.sh | INSTALL_K3S_MIRROR=cn sh -s - server \
--datastore-endpoint="mysql://username:password@tcp(hostname:3306)/database-name"
```

查看K3s集群是否创建成功

```bash

# 执行以下命令，看大两个具有master角色节点表示集群正常
sudo k3s kubectl get nodes
 
sudo k3s kubectl get pods --all-namespaces

```

保存并使用 kubeconfig 文件

```bash
cp /etc/rancher/k3s/k3s.yaml  ~/.kube/config/

```

在这个 kubeconfig 文件中，server参数为 localhost。您需要手动更改这个地址为负载均衡器的 DNS，并且指定端口 6443。（Kubernetes API Server 的端口为 6443，Rancher Server 的端口为 80 和 443。）以下是一个示例k3s.yaml

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: [CERTIFICATE-DATA]
    server: [LOAD-BALANCER-DNS]:6443 # 编辑此行
  name: default
contexts:
- context:
    cluster: default
    user: default
  name: default
current-context: default
kind: Config
preferences: {}
users:
- name: default
  user:
    password: [PASSWORD]
    username: admin

```

您现在可以使用kubectl来管理您的 K3s 集群，例如查看Pod和容器状况

```bash
sudo kubectl get pods --all-namespaces

```

# 部署 Rancher Server

为 Rancher 创建 Namespace

```bash
kubectl create namespace cattle-system
```

安装Rancher，在集群外部的负载均衡器上终止SSL/TLS通信，使用 --set tls=external选项

```bash
helm install rancher rancher-<CHART_REPO>/rancher \
 --namespace cattle-system \
 --set tls=external
```

验证Rancher是否安装成功

```bash
kubectl -n cattle-system rollout status deploy/rancher
Waiting for deployment "rancher" rollout to finish: 0 of 3 updated replicas are available...
deployment "rancher" successfully rolled out

```

如果看到以下错误： error: deployment "rancher" exceeded its progress deadline, 您可以通过运行以下命令来检查 deployment 的状态：

```bash
kubectl -n cattle-system get deploy rancher
NAME DESIRED CURRENT UP-TO-DATE AVAILABLE AGE
rancher 3 3 3 3 3m
```

DESIRED和AVAILABLE应该显示相同的个数。

安装完成后可以打开浏览器访问Rancher Server。