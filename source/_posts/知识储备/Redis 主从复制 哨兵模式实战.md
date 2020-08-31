---
title: Redis 主从复制 哨兵模式实战
tags:
  - redis
  - 高可用
categories: 知识储备
abbrlink: '65161822'
date: 2020-08-31 17:23:00
---

# 0x00

项目中因为并发不是很高一直偷懒用的单节点 Redis ，但是有很多大 key 写入的场景，这样会影响读性能，于是准备做主从复制，顺便做一下哨兵模式。

# Redis 主从复制配置

这里很简单，只需要配置`slaveof <masterip> <masterport>`参数即可实现。

> 注： Redis 5.0 版本后建议使用`replicaof`代替`slaveof`。 因为奇妙的`政治正确`原因，迫使作者修改了配置名，虽然`slaveof`仍能使用，但是只是暂时的兼容方案。

修改后重启 Redis 查看配置显示, 表示成功

```bash
# Replication
role: slave
master_host: 192.168.14.130
master_port: 6379
master_link_status: up
...
```

# Redis 哨兵模式配置

必要配置如下
```bash
port 26379

# 当前哨兵绑定的ip，一般为本机ip
bind 192.168.2.210

# 设置master节点为 192.168.14.130 6379 上的redis，
# 别名为redis-master，当两个哨兵同意故障转移就会执行
# 一般设置N/2+1(N为哨兵总数)
sentinel monitor redis-master 192.168.14.130 6379 2

# 认为master节点断线所需的毫秒（SDOWN）
sentinel down-after-milliseconds redis-master 5000

# 如果在 180000 毫秒内 master 节点没有恢复，则认为是真正宕机
# 当下一次检测宕机 master 节点恢复后，则并入 slave 中
sentinel failover-timeout redis-master 180000

# 当 master 宕机后，最多可以多少个节点对新 master 进行同步
# 数字越小完成故障转移的时间越长
sentinel parallel-syncs redis-master 2
```

配置成功后显示如下

```bash
$ redis-cli -p 26379 -h 192.168.14.130

192.168.14.128:26379> info Sentinel
# Sentinel
sentinel_masters: 1
sentinel_tilt: 0
sentinel_running_scripts: 0
sentinel_scripts_queue_length: 0
sentinel_simulate_failure_flags: 0
master0: name=mymaster, status=ok, address=192.168.14.130:6379, slaves=2, sentinels=3
```

## 坑点

1. 哨兵模式启动顺序为: `Master -> Slave -> Sentinel`

2. 启动后`sentinels`永远等于`1`: 

因为当时配置的`bind 127.0.0.1 192.168.14.130`，如果sentinel配置了bind参数，sentinel将获取第一个ip去检测主节点状态, 由于127.0.0.1是个回环地址，所以当bind第一个ip配置成127.0.0.1时无法连接其他机器的ip，所以配置时第一个ip不能配置为回环地址，建议redis相关的ip绑定都先写私网ip再写回环ip。


# 使用 Python 连接哨兵模式 Redis

```python
from redis.sentinel import Sentinel


class RedisSentinel:
    def __init__(self, sentinel_list, name="mymaster", password="", db=0):
        self.sentinel = Sentinel(sentinel_list, socket_timeout=60)
        self.name = name
        self.password = password
        self.db = db
        
    def get_master_and_slave_conn(self):
        master = self.sentinel.master_for(
            service_name=self.name,
            socket_timeout=60,
            password=self.password,
            db=self.db)
        slave = self.sentinel.slave_for(
            service_name=self.name,
            socket_timeout=60,
            password=self.password,
            db=self.db
        )
        return master, slave

if __name__ == "__main__":
	sentinel_list = [
			("192.168.14.128", "26379"),
			("192.168.14.129", "26379"),
			("192.168.14.130", "26379")
		]
	mySentinel = RedisSentinel(sentinel_list)
	master_conn, slave_conn = mySentinel.get_master_and_slave_conn()

	master_conn.set('a', 123)
	print(slave_conn.get('a'))
```