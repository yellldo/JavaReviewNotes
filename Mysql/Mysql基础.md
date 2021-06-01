目录

- <a href="#1">Mysql基础架构</a>

- <a href="#2">Mysql日志系统</a>

- <a href="#3">Mysql事务</a>

- <a href="#4">Mysql索引</a>

- <a href="#5">Mysql锁</a>

  <h3 id="1">Mysql基础架构</h3>

  > Mysql 可以分为Server层和存储引擎层 

  概述

  > server

- Server层包含连接器、查询缓存、分析器、优化器、执行器等，涵盖Mysql的大多数核心服务功能，以及所有的内置函数（如日期、时间、数学和加密函数等），所有跨存储引擎的功能都在这一层实现，比如存储过程、触发器、视图等。

  > 存储引擎

- 负责数据的存储和提取

  ### 连接器

  负责跟客户端建立连接、获取权限、维护和管理连接。

- 连接命令

  ```shell
  mysql -h$ip -P$port -u$yser -p
  ```

  