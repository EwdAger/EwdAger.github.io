---
title: F**K ZZZQ - 记录那些因政治正确而改名的技术
tags:
  - Git
categories: 不知道怎么归类
abbrlink: '55916608'
date: 2021-02-03 18:56:10
---

# 0x00

请允许我首先给出一个和善而不失礼貌的微笑 :) 以纪念因为这些莫名其妙的事而浪费的时间。

> 在今年早些时候乔治-弗洛伊德（George Floyd）的惨死和BLM抗议活动之后，科技公司希望通过放弃master、slave、blacklist和whitelist等非包容性术语来表达对黑人社区的支持。


# Github


主分支从 master 改为 main

> 适用范围: 目前新建的项目已修改，老项目不做变更

# Redis

**目前的变化**
- 将 master-slave 架构的描述改为 master-replica
- 为 SLAVEOF 提供别名 REPLICAOF，所以仍然可以使用 SLAVEOF，但多了一个选项
- 保持继续使用 slave 来对 INFO 和 ROLE 进行回应，现在目前看来，这仍然是一个重大的破坏性变更

**计划中的变化**

- 编写一个 INFO 的替代品
- 在内部替换很多东西，因为技术原因，如果作了改动，许多 PR 也会无法应用，所以必须在某些地方进行大变动

> 目前的变化适用于 redis 5.0, 5.0以前不做变更，5.0以后可能还会有变更(6.0目前没有大变更)

# Python

master process 变更为 parent process

> 适用范围: python 3.8及其以上

