## 介绍

> 监控后台的**预警者**负责持续检查主服务器（master主机）的运行状态。一旦发现主服务器出现故障，系统将启动一个自动化的切换机制。这个机制会根据预设的投票规则，从多个备用服务器（从库）中选择一个来接替主服务器的角色。被选中的备用服务器将升级为新的主服务器，并继续无缝地对外提供服务，确保整个系统的持续稳定运行。这种机制有效避免了单点故障带来的风险，并提升了系统的整体可靠性和容错能力。



##  架构

> 哨兵集群独立于 Redis 集群，哨兵之间彼此建立连接，共同监控、管理所有的 Redis 节点。



![Redis Sentinel Architecture.png](../pics/Redis Sentinel Architecture.png)

## 作用

- **监控**

  > 监控所有 Redis 节点的状态。

- **故障转移**

  > 当哨兵发现主节点下线时，会在所有从节点中选择一个作为新的主节点，并将所有其他节点的 Master 指向新的主节点。同时已下线的原主节点也会被降级为从节点，并修改配置将 Master 指向新的主节点，等到它重新上线时就会自动以从节点进行工作。

- **通知**

  > 当哨兵选举了新的主节点之后，可以通过 API 向客户端进行通知。



## 原理



### 从库发现

> 对于哨兵的配置，我们只需要配置主库的信息，哨兵在连接主库之后，会调用 `INFO` 命令获取主库的信息，再从中解析出连接主库的从库信息，再以此和其他从库建立连接进行监控。

![Redis Sentinel Principle.png](../pics/Redis Sentinel Principle.png)

INFO 中的 Replication 信息：

```shell
#Replication
role:master         
connected_slaves:2
slave0:ip=172.25.0.102,port=6379,state=online,offset=258369,lag=1
slave1:ip=172.25.0.103,port=6379,state=online,offset=258508,lag=0 
master_failover_state:no-failover 
master_replid:a4a6a7f3b2e15d9a43c01d4ba6c842539e582d6a 
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:258508 
second_repl_offset:-1 
repl_backlog_active:1 
repl_backlog_size:1048576 
repl_backlog_first_byte_offset:1 
repl_backlog_histlen:258508
```

哨兵对所有节点都会每隔 10s 发送一次 `INFO` 命令，从各节点获取 Redis 集群实时的拓扑图信息。如果新节点加入，哨兵就会去监控新的节点。



### 发布/订阅机制

哨兵们在连接同一个主库之后，是通过**发布/订阅**（pub/sub）模式来发现彼此的存在的。

> **发布/订阅**（pub/sub）是一种消息通信模式，主要的目的是解耦消息发布者和消息订阅者之间的耦合。Redis 作为一个 pub/sub server，在订阅者和发布者之间起到了消息路由的功能。订阅者可以通过 subscribe 和 psubscribe 命令从 Redis 订阅自己感兴趣的消息类型，Redis 将消息类型称为**频道**（channel）。当发布者通过 publish 命令向 Redis 发送特定类型的消息时，该频道的全部订阅者都会收到此消息。这里消息的传递是**多对多**的。**一个 client 可以订阅多个 channel，也可以向多个 channel 发送消息**。



订阅后，每个哨兵每隔 2 秒都会向 `hello` 频道发布一条携带自身信息的 hello 信息，这样哨兵就能知道其他哨兵的状态、监控的主节点和是否有新的哨兵加入：

```shell
127.0.0.1:6371> subscribe __sentinel__:hello
Reading messages... (press Ctrl-C to quit)
1) "subscribe"
2) "__sentinel__:hello"
3) (integer) 1
1) "message"
2) "__sentinel__:hello"
3) "172.25.0.202,26379,5134e342cc62ac76494c140b66b7fda80340e3a8,0,mymaster,172.25.0.101,6379,0"
1) "message"
2) "__sentinel__:hello"
3) "172.25.0.203,26379,5f5ce54a6f22f71c7d273cfb9eb14377b103d4ad,0,mymaster,172.25.0.101,6379,0"
1) "message"
2) "__sentinel__:hello"
3) "172.25.0.201,26379,4fa3486dfbaca9abc62b2976e821d18e697ab2db,0,mymaster,172.25.0.101,6379,0"
```



## 监控

> 哨兵在对 Redis 节点建立 TCP 连接之后，会周期性地发送 `PING` 命令给节点（默认是 1s），以此判断节点是否正常。如果在 `down-after-millisenconds` 时间内没有收到节点的响应，它就认为这个节点掉线了。
