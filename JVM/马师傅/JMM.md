## JMM

### 硬件层数据一致性

协议很多（数据一致性协议）

现代CPU的数据一致性实现=缓存锁+ 总线锁

读取缓存以cache line为基本单位，目前为64bytes

位于同一缓存行的两个不同数据，被两个不同CPU锁定，产生互相影响的伪共享问题

使用缓存行的对其能够提高效率

### 乱序问题

CPU为了提高指令执行效率，会再一条指令执行过程中（比如去内存读数据（慢100倍）），去同时执行另一条指令，前提是，两条指令没有依赖关系。

写操作也可以进行合并。

### 如何保证特定情况下不乱序

硬件内存屏障

> sfence: 在sfence指令前的写操作当必须在sfence指令后的写操作前完成。   
> lfence: 在lfence指令前的读操作当必须在lfence指令后的读操作前完成。  
> mfence: 在mfence指令前的读写操作当必须在mfence指令后的读写操作前完成。

> 原子指令，如X86上"lock ..."指令是一个Full Barrier，执行时会锁住内存子系统来确保执行顺序，甚至跨多个CPU。Software Locks通常使用了内存屏障或者原子指令来实现变量可见性和保持顺序执行

JVM级别如何规范（JSR133）

> LoadLoad屏障  
> LoadStore屏障  
> StoreStore屏障  
> StoreLoad屏障

#### volatile 实现细节

1. 字节码层面

   ACC_VOLATILE

2. JVM层面

   volatile内存区的读写 都加屏障

   > StoreStoreBarrier  
   > volatile写操作  
   > StoreLoadBarrier

   > LoadLoadBarrier  
   > volatile读操作  
   > LoadStoreBarrier

3. OS和硬件层面

#### synchronized 实现细节

1. 字节码层面

   ACC_SYNCHRONIZED --加在方法上。隐形标记

   -
   方法调用指令检查该方法的常量池中是否包含ACC_SYNCHRONIZED标记符，如果有，JVM要求线程在调用之前请求锁，执行线程将先获取monitor,获取成功之后会执行方法体，方法执行完再释法monitor。在方法执行期间，其它任何线程都无法再获取同一个monitor对象。

   - MonitorRecord，一个被锁住的对象会和一个MR关联，有一个Owner字段存放拥有该锁的线程的唯一标识，表示该锁被这个线程占用。

   monitorenter monitorexit --- 加在代码块上

2. JVM层面

   C C++ 调用操作系统提供的同步指令

3. OS和硬件层面

   X86 lock cmpxchg/ .... 

