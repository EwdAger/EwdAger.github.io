---
title: Helm 编排教程
tags:
  - Linux
  - docker
  - rancher2
  - k8s
  - DevOps
  - helm
categories: 云原生学习笔记
abbrlink: c57a716a
date: 2020-08-05 15:46:10
---

# Helm简介

我们知道 Kubernetes 是一个分布式的容器集群管理系统，它把集群中的管理资源抽象化成一个个 API 对象，并且推荐使用声明式的方式创建，修改，删除这些对象，每个 API 对象都通过一个 yaml 格式或者 json 格式的文本来声明。这带来的一个问题就是这些 API 对象声明文本的管理成本，每当我需要创建一个应用，都需要去编写一堆这样的声明文件。

Helm 就是用来管理这些 API 对象的工具。它类似于 CentOS 的 YUM 包管理，Ubuntu 的 APT 包管理，你可以把它理解成 Kubernetes 的包管理工具。它能够把创建一个应用所需的所有 Kubernetes API 对象声明文件组合并打包在一起。并提供了仓库的机制便于分发共享，还支持模版变量替换，，同时还有版本的概念，使之能够对一个应用进行版本的管理。

**Helm 采用客户端/服务器架构，有如下组件（概念）组成：**
- **Chart**: 就是 Helm 的一个包（package），包含一个应用所有的 Kubernetes manifest 模版，类似于 YUM 的 RPM 或者 APT 的 dpkg 文件。
- **Helm CLI**: Helm 的客户端组件，它通过 gRPC aAPI 向 tiller 发送请求。
- **Tiller**: Helm 的服务器端组件，在 Kubernetes 群集上运行，负载解析客户端端发送过来的 Chart，并根据 Chart 中的定义在 Kubernetes 中创建出相应的资源，tiller 把 release 相关的信息存入 Kubernetes 的 ConfigMap 中。
- **Repository**: 用于发布和存储 Chart 的仓库。
- **Release**: 可以理解成 Chart 部署的一个实例。通过 Chart 在 Kubernetes 中部署的应用都会产生一个唯一的 Release，即使是同一个 Chart，部署多次就会产生多个 Release。

# 安装 Helm

## Helm CLI 端的安装

1. 直接下在 Helm CLI 的二进制 [release](https://github.com/helm/helm/releases) 包
2. 解压并移动至 PATH
tar -zxvf helm-v2.0.0-linux-amd64.tgz
mv linux-amd64/helm /usr/local/bin

## Tiller 的安装

### 为 Tiller 创建 K8S 的 RBAC 角色

```yaml

apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
```

安装 Tiller, 默认使用的是 ~/.kube/config 中的 CurrentContext 来指定部署的 k8s 集群，默认安装在 namespace 为 kube-system 下，init 时可以指定很多可选参数，[更多请参考官方文档](https://docs.helm.sh/using_helm/#quickstart-guide)

> 在缺省配置下， Helm 会利用 "gcr.io/kubernetes-helm/tiller" 镜像在Kubernetes集群上安装配置 Tiller；并且利用 "https://kubernetes-charts.storage.googleapis.com" 作为缺省的 stable repository 的地址。由于在国内可能无法访问 "gcr.io", "storage.googleapis.com" 等域名，阿里云容器服务为此提供了镜像站点。

```bash
helm init -i registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.10.0 \ --stable-repo-url https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts \ --service-account tiller \ --node-selectors role=worker
```

安装完成后，可以通过 helm version 查看客户端和服务端版本

```bash
helm helm version

Client: &version.Version{SemVer:"v2.10.0", GitCommit:"9ad53aac42165a5fadc6c87be0dea6b115f93090", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.10.0", GitCommit:"9ad53aac42165a5fadc6c87be0dea6b115f93090", GitTreeState:"clean"}
```

### 升级 Tiller

```bash
export TILLER_TAG=v2.0.0-beta.1
kubectl --namespace=kube-system set image deployments/tiller-deploy tiller=gcr.io/kubernetes-helm/tiller:$TILLER_TAG
```

### 卸载 Tiller

```bash
helm reset
# 或者 kubectl delete deployment tiller-deploy --namespace kube-system
```

### Helm CLI 命令简要汇总

```bash
# 搜索可用于安装的 Chart
helm search
helm search mysql
 
# 安装一个 Chart
helm install stable/mysql
 
 
# 列出 Kubernetes 中已部署的 Chart
helm list --all
 
# helm repo 的操作
helm repo update
helm repo list
helm repo add dev https://example.com/dev-charts
 
# 创建一个 Chart，会产生一个 Chart 所需的目录结构
helm create deis-workflow
 
# 安装自定义 chart
helm inspect values stable/mysql # 列出一个 chart 的可配置项
 
helm install -f config.yaml stable/mysql # 可以将修改的配置项写到文件中通过 -f 指定并替换
helm install --set name: value stable/mysql # 也可以通过 --set 方式替换
 
# 当新版本 chart 发布时，或者当你需要更改 release 配置时，helm 必须根据现在已有的 release 进行升级
helm upgrade -f panda.yaml happy-panda stable/mariadb
 
# 删除 release
helm delete happy-panda
```

# Helm Chart 的简介

chart 就是 helm 里定一个可以在 Kubernetes 环境中部署的应用包。我们可以使用 helm create 命令去创建一个 chart 的基本骨架，它的结构如下，更多 chart 语法可以参考官方的 [chart](https://github.com/helm/charts)

其中最核心的就是 templates 这个文件夹了，里面其实就是 Kubernetes 资源描述文件的模版。模版里面的内容可以通过 values.yaml 里面的内容去渲染，同时也可以在使用 helm install --set key=value xx 部署的时候去覆盖 values.yaml 里面的默认值。

## 如何创建一个新的Charts

```bash
helm create demo
```

helm将会创建一个程序名为demo的Charts，进入demo文件夹，修改value.yaml

```yaml
replicaCount: 1
 
image:
  repository: nginx  # 修改image
  tag: stable  # 修改tag
  pullPolicy: IfNotPresent
 
imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""
 
serviceAccount:
  # Specifies whether a service account should be created
  create: true
  # The name of the service account to use.
  # If not set and create is true, a name is generated using the fullname template
  name:
 
podSecurityContext: {}
  # fsGroup: 2000
 
securityContext: {}
  # capabilities:
  #   drop:
  #   - ALL
  # readOnlyRootFilesystem: true
  # runAsNonRoot: true
  # runAsUser: 1000
 
service:
  type: ClusterIP  # 修改端口发布类型，如果需要在集群外访问，设置成 NodePort
  port: 80  # 修改端口
 
ingress:
  enabled: false  # 是否创建ingress代理记录
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: nip.io   # 域名，小技巧：设置成rancher一样的域名后缀，rancher会自动用服务名生成
      paths: # 注释默认值 []   
        - ""   # 如果enabled为 true，需要修改默认值“[]”，否则创建失败
  tls: []
  #  - secretName: chart-example-tls
  #    hosts:
  #      - chart-example.local
 
resources: {}
  # We usually recommend not to specify default resources and to leave this as a conscious
  # choice for the user. This also increases chances charts run on environments with little
  # resources, such as Minikube. If you do want to specify resources, uncomment the following
  # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
  # limits:
  #   cpu: 100m
```

## charts语法检查

```bash
helm install . --name demo --namespace <namespace> --debug --dry-run
```


## 安装 charts

```bash
helm install . --name demo --namespace <namespace>
```

## 卸载 charts

```bash
helm del --purge demo
```

# 如何设置健康检测

修改 ./template/deployment.yaml 在 conainter/ports后增加一节

```yaml
containers:
  - name: {{ .Chart.Name }}
    securityContext:
      {{- toYaml .Values.securityContext | nindent 12 }}
    image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
    imagePullPolicy: {{ .Values.image.pullPolicy }}
    ports:
      - name: http
        containerPort: 8761
        protocol: TCP
    livenessProbe:   # 存活检查
      httpGet:       # httpGet方式，可以是tcp端口方式 tcpSocket，也可以执行一段脚本 exec
        path: /      # 路径
        port: http   # 端口，和ports中定义对应
      initialDelaySeconds: 10  # 应用初始化后，多久开始检查
      periodSeconds: 10  # 检查间隔
      timeoutSeconds: 5  # 检查超时
      successThreshold: 1 # 健康阈值
      failureThreshold: 6  # 失败阈值
    readinessProbe:  # 可用检查
      httpGet:
        path: /
        port: http
      initialDelaySeconds: 10
      periodSeconds: 10
      timeoutSeconds: 5
      successThreshold: 1
      failureThreshold: 6
```

# 如何设置存储卷

新增一个 ./templates/pvc.yaml

```yaml
{{- if and .Values.persistence.enabled (not .Values.persistence.existingClaim) }}
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: {{ template "demo.fullname" . }}
  annotations:
    "helm.sh/resource-policy": keep
  labels:
    app: {{ template "demo.name" . }}
    chart: "{{ .Chart.Name }}-{{ .Chart.Version }}"
    release: "{{ .Release.Name }}"
    heritage: "{{ .Release.Service }}"
spec:
  accessModes:
    - {{ .Values.persistence.accessMode | quote }}
  resources:
    requests:
      storage: {{ .Values.persistence.size | quote }}
{{- if .Values.persistence.storageClass }}
{{- if (eq "-" .Values.persistence.storageClass) }}
  storageClassName: ""
{{- else }}
  storageClassName: "{{ .Values.persistence.storageClass }}"
{{- end }}
{{- end }}
{{- end }}
```

修改value.yaml，加入

```yaml
persistence:
  enabled: true
  ## database data Persistent Volume Storage Class
  ## If defined, storageClassName: <storageClass>
  ## If set to "-", storageClassName: "", which disables dynamic provisioning
  ## If undefined (the default) or set to null, no storageClassName spec is
  ##   set, choosing the default provisioner.  (gp2 on AWS, standard on
  ##   GKE, AWS & OpenStack)
  ##
  # storageClass: "-"
  accessMode: ReadWriteOnce
  size: 30Gi
```

修改 ./templates/deployment.yaml

`ports:`后加入

```bash

volumeMounts:
   - name: data
     mountPath: /var/lib
```

`tolerations:`后加入

```yaml

volumes:
  - name: data
  {{- if .Values.persistence.enabled }}
    persistentVolumeClaim:
      claimName: {{ .Values.persistence.existingClaim | default (include "demo.fullname" .) }}
  {{- else }}
    emptyDir: {}
  {{- end -}}
```

# 参考资料

- [Helm User Guide - Helm 用户指南](https://whmzsu.github.io/helm-doc-zh-cn/)
- [Kubernetes 包管理工具 Helm 简介](https://www.jianshu.com/p/d55e91e28f94)