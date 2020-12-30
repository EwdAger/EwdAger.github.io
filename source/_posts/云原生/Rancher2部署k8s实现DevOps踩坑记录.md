---
title: Rancher2 & K8S部署踩坑记录
tags:
  - Linux
  - docker
  - rancher2
  - k8s
  - DevOps
categories: 云原生学习笔记
abbrlink: ea5c4060
date: 2020-05-12 14:35:10
---

> 本安装教程基于**CetnOS 7**环境编写

# 安装docker

## 首先安装依赖并添加国内源

```bash
$ sudo yum install -y yum-utils device-mapper-persistent-data lvm2
$ yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

## 安装docker

```bash
$ yum clean all

$ yum makecache fastyum -y

$ yum install docker-ce
```

## 启动服务并开机自起

```bash
# 通过systemctl启动服务
$ systemctl start docker

# 开机自启动
$ systemctl enable docker

# 验证docker
$ docker version
```

## 更改docker 存储目录至空间大的磁盘(默认挂载点为`/var/lib/docker`)

```bash
# 停止docker服务
$ systemctl stop docker

# 迁移目录至空间大的挂载点
$ mv /var/lib/docker /data/docker

# 添加软链接
$ ln -s /data/docker /var/lib/docker

# 重启docker
$ systemctl start docker
```

# 安装Rancher2 & K8S集群

## 安装Rancher2

> 建议rancher server单独部署在一台机器上，不并入集群内

### 安装好docker后，启动rancher server

```bash
# 挂载目录设为/data/rancher并监听本机80/443端口，注意本机其他服务不要占用这两个端口
$ sudo docker run -d --restart=unless-stopped -v /data/rancher:/var/lib/rancher/ -p 8080:80 -p 8443:443 rancher/rancher:stable
```

### 访问安装server的主机ip，设置用户名密码，设置server url，此处需设置为即将添加入集群内的其他节点主机能访问的IP，可直接设置本机的公网IP

### 修改集群自动生成的域名后缀
![修改域名](https://s1.ax1x.com/2020/04/16/Jkfs4e.png)

## 部署K8S集群

> **注意** 请使用`v1.15`版K8S

> **注意** 在添加主机命令时，需点击显示高级选项，必须将节点机公网地址配入其中，不然后续服务仅能在集群所在内网网段访问

![部署集群](https://s1.ax1x.com/2020/04/16/Jk4iFS.png)

# 部署其他服务

## 添加内部应用商店

![添加应用商店](https://s1.ax1x.com/2020/04/16/Jk4htS.png)

## 配置kubectl

1. 在集群节点上下载Kubectl

```bash
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
$ chmod +x ./kubectl
$ sudo mv ./kubectl /usr/local/bin/kubectl
```
如果服务器上下载不了kubectl可去[github release](https://github.com/kubernetes/kubernetes/releases)页面下载最新版kubectl，再上传至服务器


2. 复制kubeconfig 

进入集群仪表盘，点击 `Kubeconfig文件` 按钮，赋值里面全部内容
![Kubeconfig](https://s1.ax1x.com/2020/04/16/Jk5VhD.png)

3. 在安装好kubectl的机器上配置kubeconfig

```bash
$ mkdir ~/.kube
$ vi ~/.kube/config
# Ctrl + V ~~~
```

> **kubectl命令** 
>	\# 查看服务状态 
>	kubectl get pod -n kafka -o wide
>	\# 查看单独pod 
>	kubectl describe pod -n kafka kafka-0

## 安装 helm

```bash
curl https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get > get_helm.sh
chmod 755 get_helm.sh
./get_helm.sh
helm init --upgrade -i registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.16.0
 
helm list
 
#如果报错，修复
kubectl create serviceaccount --namespace kube-system tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
kubectl patch deploy  --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'    
helm init --service-account tiller --upgrade
 
# 添加国内源
helm repo add stable https://burdenbear.github.io/kube-charts-mirror/
helm repo update
```

helm直接部署服务

```bash
helm install --name kong --set admin.useTLS=false,admin.servicePort=8001,admin.containerPort=8001,proxy.useTLS=false,proxy.servicePort=8000,proxy.containerPort=8000,livenessProbe.httpGet.scheme=HTTP,readinessProbe.httpGet.scheme=HTTP stable/kong

```


## 配置NFS存储类

> 请在已安装`Kubectl`并配置好`kubeconfig`的机器上执行

### 安装并配置nfs服务

```bash
# 安装NFS
$ yum install -y nfs-utils
$ yum -y install rpcbind

# 配置NFS服务端，创建数据目录
$ mkdir -p /data/nfs/k8s/
$ chmod 755 /data/nfs/k8s

# NFS配置文件设置目录信息
$ vi /etc/exports
/data/nfs/k8s/ *(async,insecure,no_root_squash,no_subtree_check,rw)

# 重启nfs服务
$ /bin/systemctl start nfs.service

# 设置开机启动
$ /bin/systemctl enable nfs.service

# 查看挂载情况
$ showmount -e
>>> Export list for k8s-test2-master:
>>> /data/nfs/k8s *
```

### 创建 ServiceAccount

找个地方创建`nfs-rbac.yaml`

```nfs-rbac.yaml
kind: ServiceAccount
apiVersion: v1
metadata:
  name: nfs-client-provisioner
---
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: nfs-client-provisioner-runner
rules:
  - apiGroups: [""]
    resources: ["persistentvolumes"]
    verbs: ["get", "list", "watch", "create", "delete"]
  - apiGroups: [""]
    resources: ["persistentvolumeclaims"]
    verbs: ["get", "list", "watch", "update"]
  - apiGroups: ["storage.k8s.io"]
    resources: ["storageclasses"]
    verbs: ["get", "list", "watch"]
  - apiGroups: [""]
    resources: ["events"]
    verbs: ["create", "update", "patch"]
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: run-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    namespace: kube-system      # 此处安装进Rancher的系统命名空间，可替换
roleRef:
  kind: ClusterRole
  name: nfs-client-provisioner-runner
  apiGroup: rbac.authorization.k8s.io
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
rules:
  - apiGroups: [""]
    resources: ["endpoints"]
    verbs: ["get", "list", "watch", "create", "update", "patch"]
---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: leader-locking-nfs-client-provisioner
subjects:
  - kind: ServiceAccount
    name: nfs-client-provisioner
    namespace: kube-system      # 此处安装进Rancher的系统命名空间，可替换
roleRef:
  kind: Role
  name: leader-locking-nfs-client-provisioner
  apiGroup: rbac.authorization.k8s.io
```

**执行 `kubectl apply -f nfs-rbac.yaml -n kube-system`**

### 部署 NFS Provisioner

找个地方创建`nfs-provisioner-deploy.yaml`

```nfs-provisioner-deploy.yaml
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nfs-client-provisioner
  labels:
    app: nfs-client-provisioner
spec:
  replicas: 1
  strategy:
    type: Recreate      #---设置升级策略为删除再创建(默认为滚动更新)
  selector:
    matchLabels:
      app: nfs-client-provisioner
  template:
    metadata:
      labels:
        app: nfs-client-provisioner
    spec:
      serviceAccountName: nfs-client-provisioner
      containers:
        - name: nfs-client-provisioner
          #---由于quay.io仓库国内被墙，所以替换成七牛云的仓库
          image: 1nj0zren.mirror.aliyuncs.com/gmoney23/nfs-client-provisioner:latest
          volumeMounts:
            - name: nfs-client-root
              mountPath: /persistentvolumes
          env:
            - name: PROVISIONER_NAME
              value: nfs-client     #--- nfs-provisioner的名称，以后设置的storageclass要和这个保持一致
            - name: NFS_SERVER
              value: 10.74.20.12   #---NFS服务器地址，和 valumes 保持一致
            - name: NFS_PATH
              value: /data/nfs/k8s      #---NFS服务器目录，和 valumes 保持一致
      volumes:
        - name: nfs-client-root
          nfs:
            server: 10.74.20.12    #---NFS服务器地址
            path: /data/nfs/k8s       #---NFS服务器目录
```
**执行`kubectl apply -f nfs-provisioner-deploy.yaml -n kube-system`**

### 部署 NFS SotageClass

找个地方创建`nfs-storage.yaml`

```nfs-storage.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-storage
  annotations:
    storageclass.kubernetes.io/is-default-class: "true" #---设置为默认的storageclass
provisioner: nfs-client    #---动态卷分配者名称，必须和上面创建的"provisioner"变量中设置的Name一致
parameters:
  archiveOnDelete: "true"  #---设置为"false"时删除PVC不会保留数据,"true"则保留数据
mountOptions:
  - hard        #指定为硬挂载方式
  - nfsvers=4   #指定NFS版本，这个需要根据 NFS Server 版本号设置
```

**执行`kubectl apply -f nfs-storage.yaml -n kube-system`**

## 部署longhorn

> 教程编写当前，Longhorn为Rancher实验性功能。如已安装NFS可不安装Longhorn。

1. 在每台节点机器上安装open-iscsi

```bash
$ yum install iscsi-initiator-utils
```

2. 在Rancher应用商店安装Longhorn即可


# 其他

## Rancher完全删除脚本

[来源](https://docs.rancher.cn/rancher2x/admin-manual/remove/#_1-%E6%89%8B%E5%8A%A8%E6%B8%85%E7%90%86%E8%8A%82%E7%82%B9)

```cleaner.sh
 # 停止服务
  systemctl  disable kubelet.service
  systemctl  disable kube-scheduler.service
  systemctl  disable kube-proxy.service
  systemctl  disable kube-controller-manager.service
  systemctl  disable kube-apiserver.service

  systemctl  stop kubelet.service
  systemctl  stop kube-scheduler.service
  systemctl  stop kube-proxy.service
  systemctl  stop kube-controller-manager.service
  systemctl  stop kube-apiserver.service

  # 删除所有容器
  docker rm -f $(docker ps -qa)

  # 删除所有容器卷
  docker volume rm $(docker volume ls -q)

  # 卸载mount目录
  for mount in $(mount | grep tmpfs | grep '/var/lib/kubelet' | awk '{ print $3 }') /var/lib/kubelet /var/lib/rancher;
  do
    umount $mount;
  done

  # 备份目录
  mv /etc/kubernetes /etc/kubernetes-bak-$(date +"%Y%m%d%H%M")
  mv /var/lib/etcd /var/lib/etcd-bak-$(date +"%Y%m%d%H%M")
  mv /var/lib/rancher /var/lib/rancher-bak-$(date +"%Y%m%d%H%M")
  mv /opt/rke /opt/rke-bak-$(date +"%Y%m%d%H%M")

  # 删除残留路径
  rm -rf  /etc/ceph \
          /etc/cni \
          /opt/cni \
          /run/secrets/kubernetes.io \
          /run/calico \
          /run/flannel \
          /var/lib/calico \
          /var/lib/cni \
          /var/lib/kubelet \
          /var/log/containers \
          /var/log/pods \
          /var/run/calico

  # 清理网络接口
  no_del_net_inter='
  lo
  docker0
  eth
  ens
  bond
  '
  network_interface=`ls /sys/class/net`
  for net_inter in $network_interface;
  do
    if ! echo "${no_del_net_inter}" | grep -qE ${net_inter:0:3}; then
      ip link delete $net_inter
    fi
  done

  # 清理残留进程
  port_list='
  80
  443
  6443
  2376
  2379
  2380
  8472
  9099
  10250
  10254
  '

  for port in $port_list;
  do
    pid=`netstat -atlnup | grep $port | awk '{print $7}' | awk -F '/' '{print $1}' | grep -v - | sort -rnk2 | uniq`
    if [[ -n $pid ]];then
      kill -9 $pid
    fi
  done

  kube_pid=`ps -ef | grep -v grep | grep kube | awk '{print $2}'`

  if [[ -n $kube_pid ]];then
    kill -9 $kube_pid
  fi

  # 清理Iptables表
  ## 注意：如果节点Iptables有特殊配置，以下命令请谨慎操作
  sudo iptables --flush
  sudo iptables --flush --table nat
  sudo iptables --flush --table filter
  sudo iptables --table nat --delete-chain
  sudo iptables --table filter --delete-chain

  systemctl restart docker
```