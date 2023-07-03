## Mq介绍

> 消息队列是一种先进先出的数据结构

## 哪些文件适合使用消息队列来解决

- 异步处理
    - 可以更快地返回结果；
    - 减少等待，自然实现了步骤之间的并发，提升系统总体的性能
- 流量控制
- 服务解耦

## 角色介绍

- Producer:消息生产者
- Consumer:消息消费者
- Broker:面向Producer和Consumer接受和发送消息
- NameServer:用来收集其他角色的信息
- Topic:区分消息的种类；一个发送者可以发送消息给一个或多个Topic；一个消息的接受者可以订阅一个或者多个Topic
- Message Queue:相当于是Topic的分区；用于并行发送和接收消息

### broker集群

- broker高可用，可以配成Master/Slave结构，Master可写可读，Slave只读，Master将写入的数据同步到Slave。
    - 一个Master可以对应多个Slave，但是一个Slave只对一个Master。
    - Master与Slave的对应关系通过执行相同的BrokerName，不同的BrokerId来定义 BrokerId为0表示为Master，非0表示Slave。
- Master多机负载，可以部署多个broker。
    - 每个broker与NameServer集群中的所有节点建立长连接，定时注册Topic信息到所有的NameServer。
- 每个broker节点，在启动时，都会遍历NameServer列表，与每个NameServer建立长连接，注册自己的信息，之后定时上报。
- 是消息中间件的消息存储，转发服务器

### producer

- 消息生产者
- 通过NameServer集群中的其中一个节点（随机选择）建立长连接，获得Topic的路由信息，包括Topic下面有哪些Queue，这些Queue分布在哪些Queue上等。
- 接下来向提供Topic服务的Master建立长连接，且定时向Master发送心跳。

### consumer

- 消息的消费者，通过NameServer集群获得Topic的路由信息，连接到对应的Broker上消费消息。
- 注意：由于Master和Slave都可以读取消息，因此Consumer会与Master和Slave都建立连接。

### NameServer

- 底层由Netty实现，提供了路由管理、服务注册、服务发现的功能，是一个无状态的节点
- NameServer是服务发现者，集群中各个角色都需要定时向NameServer上报自己的状态，以便相互发现彼此，超时不上报的话，NameServer会把它从列表中剔除掉。
- NameServer可以部署多个，当多个NameServer存在时，其他角色同时向多个NameServer上报信息，以保证高可用。
- NameServer集群间互不通信，没有主备的概念。
- NameServer中的Broker、Topic等信息默认不持久化。
