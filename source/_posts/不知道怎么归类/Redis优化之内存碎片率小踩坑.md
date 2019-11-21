---
title: Redis优化之内存碎片小踩坑
tags: Redis
categories: 不知道怎么归类
abbrlink: 9779499d
date: 2019-11-21 10:34:00
---

<script type="text/javascript"
   src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>


## 起因

近来项目里引用了Redis，本来用这高端玩意就是为了加速项目跑的速度。然而，用着用着越来越慢。而之前就做着性能优化的活，也顺手接下了优化Redis的活

## 内存碎片率`mem_fragmentation_ratio`

查阅相关资料得知，速度过慢很有可能是因为内存不足使用了`swap`导致。而`mem_fragmentation_ratio`是一个很明显的是否使用了`swap`的指标。

`mem_fragmentation_ratio`的计算公式为

$$MemFragmentationRatio = \frac {UsedMemoryRss} {UsedMemory}$$


而我Redis执行info命令结果为：

```bash
# Memory
used_memory:3803742104
used_memory_human:3.54G
used_memory_rss:3531386880
used_memory_rss_human:3.29G
used_memory_peak:3940788176
used_memory_peak_human:3.67G
total_system_memory:33567162368
total_system_memory_human:31.26G
used_memory_lua:37888
used_memory_lua_human:37.00K
maxmemory:21474836480
maxmemory_human:20G
maxmemory_policy:noeviction
mem_fragmentation_ratio:0.93
mem_allocator:jemalloc-3.6.0
```

而相关资料显示，ratio值正常范围在1\~1.5左右。
- 大于1.5表示，系统分配的内存大于Redis实际使用的内存，Redis没有把这部分内存返还给系统，产生了很多内存碎片。在Redis 4.0版以前，只能通过安全重启解决这个问题。
- 小于1表示，系统分配的内存小于Redis实际使用的内存，而Redis很有可能在使用Swap了！使用swap是相当影响性能的。


而我这个ratio小于1，那么说明很有可能使用Swap了。

## 初步尝试

```bash
$ free -h
total        used        free      shared  buff/cache   available
Mem:            31G        5.6G        326M         49M         25G         25G
Swap:            0B          0B          0B
```

woc,没有使用Swap？那是什么鬼，那为啥我的ratio还小于1？

## 死马当活马医

然后尝试ratio大于1的方法，重启大法尝试。然后发现日志

```bash
(error) MISCONF Redis is configured to save RDB snapshots, but is currently not able to persist on disk. Commands that may modify the data set are disabled. Please check Redis logs for details about the error
```

噫？竟然没有写持久化文件RDB的权限，根据Redis配置文件查询到RDB位置，后访问发现文件夹权限为

```bash
drwxrwxr-x  2 hdfs  hdfs   36 Oct 29 18:56 analysis_flink
drwxrwxr-x 11 hdfs  hdfs  286 Nov 16 14:24 flink-1.9.0
drwxrwxr-x  8 hdfs  hdfs  155 Nov  2 18:18 kafka-manager
drwxr-xr-x 10 hdfs  hdfs  203 Sep 25 11:47 kafka-manager-2.0.0.2
drwxrwxrwx  3 hdfs  hdfs   23 Sep 16 21:11 local
drwxr-xr-x  2 root root  61 Nov 21 11:22 redis_data
drwxrwxr-x  3 hdfs  hdfs   81 Oct 26 16:39 yhc_spark
```

好吧，权限竟然是root，实际应该`chown`为redis用户及用户组。

然后Redis就能正常关闭了。

可是，在Redis-cli中执行`info memory`，ratio值仍然为0.93

手动微笑 ：）

## 无穷无尽的谷歌

不得不佩服CSDN的污染搜索引擎的实力，中文搜索Google第一页几乎都是互相抄。经过各种奇巧淫技终于在Github issue中找到原因.

 [Fragmentation ratio < 1 but not swapping. #946](https://github.com/antirez/redis/issues/946)

> *antirez commented on 15 Feb 2013*
Hello, in your case used physical memory appears to be smaller than virtual memory but you report no swapped pages. It is likely that in your data set there are many blank pages (all filled with zero) so multiple pages are actually mapped to the same zero page.<br>
Anyway this is never going to be an issue, the worst condition may be an error in the way Redis reports memory, but will have no effects in the stability and functionality of the server.<br>
Thanks for reporting, closing because it is not a bug but just an effect of OS memory management.

翻译过来大意就是数据集中有很多全部填充为零的数据，他们会全部映射到同一个内存区域。所以会导致实际使用内存大于系统分配的内存！

综上所述，`mem_fragmentation_ratio`只是一个**可能**反应性能指标的东西，而并不完全能反应性能指标。

## 那为啥我的Redis这么慢

![网络延迟测试](https://gvoidy-1251878576.cos.ap-chengdu.myqcloud.com/redis%E6%85%A2)

emm...本机连接远程Redis服务器的速度大约`3M/s`。反观项目中存储方式，是将一个巨大的存有数据集的对象直接使用`pickle`序列化成一个巨大的字符串存进Redis中的....所以Redis中一个Key大小为`20~30M`不慢才怪。

最后只好老老实实优化代码，改存为`Hash`格式，每次只存取需要的东西，果然速度提升了很多。