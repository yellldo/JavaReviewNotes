### Ribbon

- Ribbon是通过ILoadBalancer接口对外提供统一的选择服务器（Server）的功能，此接口会根据不同的负载均衡策略（IRule）选择合适的Server返回给使用者。
- IRule是负载均衡策略的抽象，ILoadBalancer通过调用IRule的choose方法返回Server。
- IPing用来检测Server是否可用，ILoadBalancer的实现类维护一个Time每隔10S检测一次Server的可用状态。
- IClientConfig主要定义了用于初始化各种客户端和负载均衡器的配置信息，其实现类为DefalutClientConfigImpl。

>
Ribbon的负载均衡策略分为五种RandomRule、RoundRobinRule、RetryRule、WeightResponseTimeRule、ClientConfigEnabledRoundRobinRule、BestAvailableRule、ZoneAvoidanceRule、AvailablityFilterRule、

- RondomRule

  > 随机选择一个实例

- RoundRobinRule

  >
  轮训，整体逻辑：开启一个计数器count，在while循环中遍历服务清单，获取清单之前先通过incrementAndGetModulo方法获取一个下标，这个下标就是一个不断增长的数先加1然后和服务清单总数取模之后获取到的（所以这个下标不会越界），拿着下标再去服务清单中取服务，每次轮训计数器都会加1，如果连续10次都没有取到服务，则会报一个警告No
  available alive servers after 10s tries from load balance xxx.

- RetryRule

  > 在轮训的基础上重试

- WeightResponseTimeRule

  > 权重，

- ClientConfigEnabledRoundRobinRule

  > 逻辑和RoundRobinRule是一样的，

- BestAvailableRule

  > 继承于ClientConfigEnabledRoundRobinRule，在它的基础上增加了根据loadBalancerStats中保存的服务实例的状态信息来过滤掉失效服务实例的功能，然后顺便找出并发请求最小的服务实例来使用。

- ZoneAvoidanceRule

  > 默认规则，复合判断Server所在区域的性能和Server的可用性选择服务器。

- AvailablityFilterRule

  > 先过滤掉故障实例，再选择并发较小的实例。