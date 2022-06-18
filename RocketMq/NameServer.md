## NameServer

> NameServer是一个独立的进程，为Broker、生产者和消费者提供服务。
>
> NameServer最主要的功能就是，为客户端提供寻址服务，协助客户端找到主题对应的Broker地址。
>
> 此外，NameServer还负责监控每个Broker的存活状态。

- 每个Broker都需要和所有的NameServer节点进行通信（保持长连接）。当Broker保存的Topic信息发生变化时，它会主动通知所有NameServer更新路由信息，为了保证数据一致性。
- Broker每隔30s所有的NameServer节点上报路由信息。这个上报路由信息的RPC请求，也同时起到Broker和NameServer之间的心跳作用，NameServer依靠这个心跳来确定Broker的健康状态。
- NameServer每隔10s会扫描 一次brokerLiveTable，如果120s内没有收到心跳，就认为broker失效，并更新topic路由信息，将失效的broker剔除掉。

NameServer结构非常简单，排除 KV 读写相关的类之后，一共只有 6 个类。

- **NameStartUp**：程序入口
- **NamesrvController**：NameServer 的总控制器，负责所有服务的生命周期管理。
- **RouteInfoManager**：NameServer最核心的实现类，负责保存和管理集群路由信息。
- **BrokerHousekeepingService**：监控 Broker 连接状态的代理类。
- **DefaultRequestProcessor**：负责处理客户端和 Broker 发送过来的 RPC 请求的处理器。
- **ClusterTestRequestProcessor**：用于测试的请求处理器。（测试类不需要管）


1. 一个Topic默认有4个queue
2. 一个Broker默认为每一个主题创建4个读队列和4个写队列