### Redis基本数据类型

> String、Sets、Sorted Sets、List、Hashes

- Redis除了这五种基本数据类型，还有Bitmaps、HyperLogLogs、Streams。

### Redis过期策略

> 定期删除、惰性删除

- 定期删除

  > Redis默认是每隔100ms就随机抽取一些设置了过期时间的key，检查是否过期，如果过期了就删除。

  #### 问题

  因为定期删除是随机扫描，可能会导致有很多过期的key没有被删除。那就会走惰性删除。

- 惰性删除

  > 在你获取某个key的时候，Redis会检查一下，这个key如果设置了过期时间那么是否过期，过期了就删除不会返回任何东西。

  #### 问题

  如果定期删除漏掉很多key，也没有走惰性删除。如果大量过期key堆积在内存中，导致Redis内存耗尽了。这个时候就要走**内存淘汰机制**了。

### Redis内存淘汰机制

> noeviction：当内存不足以容纳新写入数据时，新写入操作会报错，这个一般没人用吧，实在是太恶心了。  
> allkeys-lru：当内存不足以容纳新写入数据时，在**键空间**中，移除最近最少使用的 key（这个是**最常用**的）。  
> allkeys-random：当内存不足以容纳新写入数据时，在**键空间**中，随机移除某个 key，这个一般没人用吧，为啥要随机，肯定是把最近最少使用的 key 给干掉啊。  
> volatile-lru：当内存不足以容纳新写入数据时，在**设置了过期时间的键空间**中，移除最近最少使用的 key（这个一般不太合适）。  
> volatile-random：当内存不足以容纳新写入数据时，在**设置了过期时间的键空间**中，**随机移除**某个 key。  
> volatile-ttl：当内存不足以容纳新写入数据时，在**设置了过期时间的键空间**中，有**更早过期时间**的 key 优先移除。

### 对象的类型与编码

> Redis的每个对象都有一个redisObject结构表示，该结构中和保存数据有关的三个属性分别是type、encoding、ptr。

```c
typedef struct redisObject {
    // 类型
    unsigned type 4;
    // 编码
    unsigned encoding 4;
    // 指向底层实现数据结构的指针
    unsigned *ptr;
}robj;
```

#### 对象类型

|   常量类型   |  对象的名称  |
| :----------: | :----------: |
| REDIS_STRING |  字符串对象  |
|  REDIS_LIST  |   列表对象   |
|  REDIS_HASH  |   哈希对象   |
|  REDIS_SET   |   集合对象   |
|  REDIS_ZSET  | 有序集合对象 |

