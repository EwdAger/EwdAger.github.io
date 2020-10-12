---
title: Spark vs Dask Python生态下的计算引擎
tags:
  - Python
  - 机器学习
  - 计算引擎
  - spark
  - dask
abbrlink: 1b5a6244
categories: 机器学习笔记
date: 2020-09-22 10:05:00
---

> 本文基于[Gurpreet Singh](https://databricks.com/speaker/gurpreet-singh)大佬在 Spark+AI SUMMIT 2020 的[公开课](https://databricks.com/session/dask-and-apache-spark)编写

# 0x00

对于 Python 环境下开发的数据科学团队，Dask 为分布式分析指出了非常明确的道路，但是事实上大家都选择了 Spark 来达成相同的目的。Dask 是一个纯 Python 框架，它允许在本地或集群上运行相同的 Pandas 或 Numpy 代码。而 Spark 即时使用了 Apache 的 pySpark 包装器，仍然带来了学习门槛，其中涉及新的 API 和执行模型。鉴于以上陈述，我们下面将对比这两个技术方案。

# Spark vs Dask

首先先上Dask和Spark的架构设计图~
![设计架构](https://s1.ax1x.com/2020/09/22/wL8Tbt.png)

## 生态
Dask 对于 Python 生态中的 Numpy、Pandas、Scikit-learn等有很好的兼容性，并且在`low level api`中提供了延迟执行的方法。

Spark 是独立于 Python 生态的另一个项目，但如果是在 JVM 环境下开发，并且十分需要使用 Spark SQL 等特性，可以考虑使用Spark。

## 性能

Dask 中的 dataframe 基本上由许多个 pandas 的 dataframe 组成，他们称为分区。但是因为 Dask 需要支持分布式，所以有很多 api 不完全和 pandas 中的一致。并且在涉及到排序、洗牌等操作时，在 pandas 中很慢，在 dask 中也会很慢。除此之外，dask 几乎都是遵循 pandas 设计的。

Spark 因为他依赖于 JVM ，在性能方面是有很多优势的，但是如果我们使用 pySpark ，提交任务和获得结果需要`Python - JVM`、`JVM - Python`之间的转换、上下文绑定等操作。而这些操作是很耗时且有峰值的。

> PySpark 采用了 Python、JVM 进程分离的多进程架构，在 Driver、Executor 端均会同时有 Python、JVM 两个进程。当通过 spark-submit 提交一个 PySpark 的 Python 脚本时，Driver 端会直接运行这个 Python 脚本，并从 Python 中启动 JVM；而在 Python 中调用的 RDD 或者 DataFrame 的操作，会通过 Py4j 调用到 Java 的接口。在 Executor 端恰好是反过来，首先由 Driver 启动了 JVM 的 Executor 进程，然后在 JVM 中去启动 Python 的子进程，用以执行 Python 的 UDF，这其中是使用了 socket 来做进程间通信。

## 对于机器学习的支持

Dask 原生支持 Scikit-learn，并且将某些 Scikit-learn 中的方法重构改成了分布式的方式。并且可以轻易兼容 Python 生态中的开源算法包。并且可以通过 Dask 提供的`延迟执行装饰器`使用 Python 编写支持分布式的自定义算法。

Spark 中也有Spark-mllib 可以高效的执行编写好的机器学习算法，而且可以使用在spark worker上执行sklearn的任务。能兼容 JVM 生态中开源的算法包。并且可以通过 UDF 执行使用 Python 编写的自定义算法。

## 对于深度学习的支持

Dask 直接提供了方法执行 tensorflow，而tensorflow本身就支持分布式。

目前pySpark缺少开源的深度学习框架，目前有兼容主流python社区深度学习框架的项目，但目前处于实验阶段还不成熟

# 编码层的考虑因素

- APIs
	- 自定义算法（Dask）
	- SQL, Graph (pySpark)

- Debug
	- dask分布式模式不支持常用的python debug工具
	- pySpark的error信息是jvm、python混在一起报出来的

- 可视化
	- 将大数据集抽样成小数据集，再用pandas展示
	- 使用开源的D3、Seaborn、DataShader等（Dask)框架
	- 使用 databircks 可视化特性

## 选择 Spark 的原因

- 你更喜欢 Scala 或使用 SQL 
- 你是基于或者更偏向 JVM 生态的开发
- 你需要一个更成熟、更值得信赖的解决方案
- 你大部分时间都在用一些轻量级的机器学习进行商业分析
- 你想要一个一体化的解决方案

## 选择 Dask 的原因

- 你更喜欢 Python 或本地运行，或者不希望完全重写遗留的 Python 项目
- 你的用例很复杂，或者不完全适合 Spark 的计算模型（MapReduce）
- 你只希望从本地计算过渡到集群计算，而不用学习完全不同的语言生态
- 你希望与其他 Python 生态的技术结合，并且不介意多安装几个包

# 总结

- Spark 是一个成熟的、包罗万象的方案。如果你已经在使用大数据集群，且需要一个能做所有事情的项目，那么 Spark 是一个很好的选择，特别是你的用例是典型的 ETL + SQL，并且你在使用 Scala 编写程序。

- Dask 更轻量、更容易集成到现有的代码里。如果你的问题超出了典型的 ETL + SQL，并且你希望为现有的解决方案添加灵活的并行性，那么 Dask 可能是一个更好的选择，特别是你已经在使用 Python相关的库，比如 Numpy 和 Pandas 时。