# Executor

# ExecutorService

AbstractExecutor

# ThreadPoolExecutor

1. corePoolSize 核心线程数

2. maximumPoolSize 最大线程数

3. keepAliveTime 闲置时间

4. TimeUnit 时间单位

5. BlockingQueue 任务队列

6. ThreadFactory 线程工程

7. RejectStrategy

   > Abort 抛出异常
   >
   > DisCard 抛弃掉
   >
   > DisCardOld 抛弃掉老的
   >
   > CallerRuns 调用者执行