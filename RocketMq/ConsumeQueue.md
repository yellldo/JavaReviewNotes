## ConsumeQueue

> consumequeue的第一级目录为消息主题，第二级目录为主题的消息队列
>
> 单个ConsumeQueue文件中默认包含30万个条目，单个文件的长度为30w×20字节，单个ConsumeQueue文件可以看出是一个ConsumeQueue条目的数组，其下标为Consume-Queue的逻辑偏移量，消息消费进度存储的偏移量即逻辑偏移量。  
>
> ConsumeQueue即为Commitlog文件的索引文件，其构建机制是当消息到达Commitlog文件后，由专门的线程产生消息转发任务，从而构建消息消费队列文件与下文提到的索引文件。

  

