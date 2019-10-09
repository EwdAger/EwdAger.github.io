---
title: 聚类算法之K-Means(K均值)聚类
tags:
  - Python
  - 机器学习
  - 聚类算法
abbrlink: dea808f4
categories: 机器学习笔记
date: 2019-10-09 14:45:00
---

## 聚类与分类的区别

**分类**： 类别是已知的，通过对已知分类的数据进行训练和学习，找到不同类的特征再对未分类的数据进行分类，属于监督学习。

**聚类**： 事先不知道数据会分为几类，通过聚类分析将数据聚合成几个群体。聚类不需要对数据进行训练和学习。属于无监督学习。

*数据输入有标签则为监督学习，否则为无监督学习*

## k-means 聚类怎么算

1. 首先输入 k 的值，即希望通过聚类得到 k 个分组；
2. 从数据集中随机选取 k 个数据点作为初始质心
3. 对数据中每一个点计算与每一个质心的距离，离哪个质心近，就分为哪一类
4. 基于当前分好的类，重新计算类中的所有向量的平均值，确定新的质心
5. 重复3、4步
6. 直到新的质心与老的质心的距离小于某一个设定的阈值，可以认为达到预期，算法终止。

<!--more-->

具体步骤如下图：
![k-means](https://gvoidy-1251878576.cos.ap-chengdu.myqcloud.com/k-means)

## k-means 聚类 Python 实现

```python
import numpy as np
from sklearn.cluster import KMeans
data = np.random.rand(100, 3) #生成一个随机数据，样本大小为100, 特征数为3

#假如我要构造一个聚类数为3的聚类器
estimator = KMeans(n_clusters=3)#构造聚类器
estimator.fit(data)#聚类
label_pred = estimator.labels_ #获取聚类标签
centroids = estimator.cluster_centers_ #获取聚类中心
inertia = estimator.inertia_ # 获取聚类准则的总和
```

## 主函数 KMeans 参数解释

```python
sklearn.cluster.KMeans(n_clusters=8,
	init='k-means++', 
	n_init=10, 
	max_iter=300, 
	tol=0.0001, 
	precompute_distances='auto', 
	verbose=0, 
	random_state=None, 
	copy_x=True, 
	n_jobs=1, 
	algorithm='auto'
	)
```

- `n_clusters`:簇的个数，即你想聚成几类
- `init`: 初始簇中心的获取方法
- `n_init`: 获取初始簇中心的更迭次数，为了弥补初始质心的影响，算法默认会初始10次质心，实现算法，然后返回最好的结果。
- `max_iter`: 最大迭代次数（因为kmeans算法的实现需要迭代）
- `tol`: 容忍度，即kmeans运行准则收敛的条件
- `precompute_distances`：是否需要提前计算距离，这个参数会在空间和时间之间做权衡，如果是True 会把整个距离矩阵都放到内存中，auto 会默认在数据样本大于featurs*samples 的数量大于12e6 的时候False,False 时核心实现的方法是利用Cpython 来实现的
- `verbose`: 冗长模式（不太懂是啥意思，反正一般不去改默认值）
- `random_state`: 随机生成簇中心的状态条件。
- `copy_x`: 对是否修改数据的一个标记，如果True，即复制了就不会修改数据。bool 在scikit-learn 很多接口中都会有这个参数的，就是是否对输入数据继续copy 操作，以便不修改用户的输入数据。这个要理解Python 的内存机制才会比较清楚。
- `n_jobs`: 并行设置
- `algorithm`: kmeans的实现算法，有：`auto`, `full`, `elkan`, 其中 `full`表示用EM方式实现

## 具体 Python 实现

```python
# coding=utf-8
"""
Created on 2019/10/9 16:31

@author: EwdAger
"""
import numpy as np
import pandas as pd
from sklearn.cluster import KMeans
import matplotlib.pyplot as plt

data = pd.read_csv(r"F:\MachineLearning\Kmeans\data\testSet.txt", sep='\t',
                   header=0, dtype=str, na_filter=False).astype(np.float)

k = 4
clt = KMeans(n_clusters=k)
clt.fit(data)
cents = clt.cluster_centers_  # 获取聚类中心
labels = clt.labels_  # 获取聚类标签
colors = ['b', 'g', 'r', 'k', 'c', 'm', 'y', '#e24fff', '#524C90', '#845868']

# 画图
for i in range(k):
    # 获取i组数据的index
    index = np.nonzero(labels == i)[0]
    x0 = data.iloc[data.index.isin(index), 0].tolist()
    x1 = data.iloc[data.index.isin(index), 1].tolist()
    y_i = i
    # 给第i组数据涂上颜色，进行分组
    for j in range(len(x0)):
        plt.text(x0[j], x1[j], str(y_i), color=colors[i], fontdict={'weight': 'bold', 'size': 6})
    # 绘制该组质心
    plt.scatter(cents[i, 0], cents[i, 1], marker='x', color=colors[i], linewidths=7)

plt.axis([-7, 7, -7, 7])
plt.show()

```

结果如下：
![k_clusters4](https://gvoidy-1251878576.cos.ap-chengdu.myqcloud.com/k_clusters4.png)