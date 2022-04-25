## RouteInfoManager

> 保存了所有的路由信息，这些路由信息都保存在内存中，并没有持久化。

```java
// 保存主题和队列的信息。QueueData保存的Broker name
private final HashMap<String/* topic */, List<QueueData>> topicQueueTable;
private final HashMap<String/* brokerName */, BrokerData> brokerAddrTable;
private final HashMap<String/* clusterName */, Set<String/* brokerName */>> clusterAddrTable;
private final HashMap<String/* brokerAddr */, BrokerLiveInfo> brokerLiveTable;
private final HashMap<String/* brokerAddr */, List<String>/* Filter Server */> filterServerTable;
```

### 路由分发器

```java
public TopicRouteData pickupTopicRouteData(final String topic) {
 
    // 初始化返回数据 topicRouteData
    TopicRouteData topicRouteData = new TopicRouteData();
    boolean foundQueueData = false;
    boolean foundBrokerData = false;
    Set<String> brokerNameSet = new HashSet<String>();
    List<BrokerData> brokerDataList = new LinkedList<BrokerData>();
    topicRouteData.setBrokerDatas(brokerDataList);
 
    HashMap<String, List<String>> filterServerMap = new HashMap<String, List<String>>();
    topicRouteData.setFilterServerTable(filterServerMap);
 
    try {
        try {
 
            // 加读锁
            this.lock.readLock().lockInterruptibly();
 
            // 先获取主题对应的队列信息
            List<QueueData> queueDataList = this.topicQueueTable.get(topic);
            if (queueDataList != null) {
 
                // 把队列信息返回值中
                topicRouteData.setQueueDatas(queueDataList);
                foundQueueData = true;
 
                // 遍历队列，找出相关的所有 BrokerName
                Iterator<QueueData> it = queueDataList.iterator();
                while (it.hasNext()) {
                    QueueData qd = it.next();
                    brokerNameSet.add(qd.getBrokerName());
                }
 
                // 遍历这些 BrokerName，找到对应的 BrokerData，并写入返回结果中
                for (String brokerName : brokerNameSet) {
                    BrokerData brokerData = this.brokerAddrTable.get(brokerName);
                    if (null != brokerData) {
                        BrokerData brokerDataClone = new BrokerData(brokerData.getCluster(), brokerData.getBrokerName(), (HashMap<Long, String>) brokerData
                            .getBrokerAddrs().clone());
                        brokerDataList.add(brokerDataClone);
                        foundBrokerData = true;
                        for (final String brokerAddr : brokerDataClone.getBrokerAddrs().values()) {
                            List<String> filterServerList = this.filterServerTable.get(brokerAddr);
                            filterServerMap.put(brokerAddr, filterServerList);
                        }
                    }
                }
            }
        } finally {
            // 释放读锁
            this.lock.readLock().unlock();
        }
    } catch (Exception e) {
        log.error("pickupTopicRouteData Exception", e);
    }
 
    log.debug("pickupTopicRouteData {} {}", topic, topicRouteData);
 
    if (foundBrokerData && foundQueueData) {
        return topicRouteData;
    }
 
    return null;
}
```

