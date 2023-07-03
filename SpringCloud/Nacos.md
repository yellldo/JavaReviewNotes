## Nacos

### 服务注册

> 1. 初次注册，会创建一个空的Service 为后面做准备
>
> 2. 添加实例
> 3. 根据namespaceId、serviceName、ephemeral 构建key
> 4. onPut 将注册实例更新到内存注册表

```java
 public void registerInstance(String namespaceId, String serviceName, Instance instance) throws NacosException {
        // 创建一个空的Service
        createEmptyService(namespaceId, serviceName, instance.isEphemeral());
        
        Service service = getService(namespaceId, serviceName);
        
        if (service == null) {
            throw new NacosException(NacosException.INVALID_PARAM,
                    "service not found, namespace: " + namespaceId + ", service: " + serviceName);
        }
        // 服务实例添加到对应的service列表中，并创建一个健康检查的task
        addInstance(namespaceId, serviceName, instance.isEphemeral(), instance);
    }
```

```java
public void addInstance(String namespaceId, String serviceName, boolean ephemeral, Instance... ips)
            throws NacosException {
		// 构建key
        String key = KeyBuilder.buildInstanceListKey(namespaceId, serviceName, ephemeral);

        Service service = getService(namespaceId, serviceName);

        synchronized (service) {
            List<Instance> instanceList = addIpAddresses(service, ephemeral, ips);

            Instances instances = new Instances();
            instances.setInstanceList(instanceList);
			
            consistencyService.put(key, instances);
        }
    }
```

### 客户端注册

> 
> 心跳检测任务获取目标服务端下标 int target = distroHash(serviceName) % servers.size(); 
