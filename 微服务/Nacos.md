## Nacos

> Nacos 1.x版本调用接口是通过Http的方式，2.x版本改为rpc

### 服务注册

> 客户端服务注册入口是NacosNamingService.registerInstance

```java
public void registerInstance(String serviceName, String groupName, String ip, int port, String clusterName)
            throws NacosException {
        // 组装参数
        Instance instance = new Instance();
        instance.setIp(ip);
        instance.setPort(port);
        instance.setWeight(1.0);
        instance.setClusterName(clusterName);
        // 向服务端/nacos/v1/ns/instance 发送请求
        registerInstance(serviceName, groupName, instance);
    }
```

> 服务器端收到请求

```java
public void registerInstance(String namespaceId, String serviceName, Instance instance) throws NacosException {
        // 如果是第一次注册，就创建一个空的service，不是则将注册的服务put到serviceMap中 并初始化（创建一个心跳检测的task）
        createEmptyService(namespaceId, serviceName, instance.isEphemeral());
        // 获取刚刚创建的service
        Service service = getService(namespaceId, serviceName);
        
        if (service == null) {
            throw new NacosException(NacosException.INVALID_PARAM,
                    "service not found, namespace: " + namespaceId + ", service: " + serviceName);
        }
        // 服务实例添加到对应的service列表中，并创建一个健康检查的task
        addInstance(namespaceId, serviceName, instance.isEphemeral(), instance);
    }
```

