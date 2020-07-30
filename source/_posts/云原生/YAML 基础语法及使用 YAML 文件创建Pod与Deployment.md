---
title: k8s中服务添加hosts及一键转换脚本
tags:
  - Linux
  - docker
  - k8s
categories: 云原生学习笔记
abbrlink: 65fb69b
date: 2020-05-12 14:35:10
---

# 序


项目管理k8s集群用的是rancher，可是rancher没有提供给deployment批量添加hosts的图形化界面，所以还是只能按照k8s官方的方法修改yaml文件。 

# 示例

[使用 HostAliases 向 Pod /etc/hosts 文件添加条目](https://kubernetes.io/zh/docs/concepts/services-networking/add-entries-to-pod-etc-hosts-with-host-aliases/)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostaliases-pod
spec:
  restartPolicy: Never
  hostAliases:
  - ip: "127.0.0.1"
    hostnames:
    - "foo.local"
    - "bar.local"
  - ip: "10.1.2.3"
    hostnames:
    - "foo.remote"
    - "bar.remote"
  containers:
  - name: cat-hosts
    image: busybox
    command:
    - cat
    args:
    - "/etc/hosts"
```

# 一键转换脚本

> 本脚本仍需要一定的手动操作

- 将需要修改的hosts文件改为csv格式（记得设置表头）

- 执行脚本后需要去掉多余的`'`

```python
import pandas as pd
import yaml

data = pd.read_csv(r"hosts.csv")

ip = data.loc[:, "ip"].tolist()
hosts = data.loc[:, "hosts"].tolist()

res = {
    "hostAliases": []
}

for i in range(len(ip)):
    res_side = {
        "ip": f"\"{ip[i]}\"",
        "hostnames": [f"\"{hosts[i]}\""]
    }
    res['hostAliases'].append(res_side)

with open('hosts.yaml', 'w') as f:
    yaml.dump(res, f)
    f.close()

```