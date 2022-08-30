### CMS

> CMS是老年代垃圾收集器 通过`-XX:+UseConcMarkSweepGC`来开启CMS。

### 垃圾回收

- 初始标记（STW）
- 并发标记（CMS线程和用户线程同时运行）
- 重新标记（STW）
- 并发清理（CMS线程和用户线程同时运行）
- 并发重置（CMS线程和用户线程同时运行）