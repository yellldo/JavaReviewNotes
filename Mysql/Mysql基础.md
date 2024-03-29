## Mysql基础架构

> Mysql 可以分为Server层和存储引擎层

### 概述

> server

- Server层包含连接器、查询缓存、分析器、优化器、执行器等，涵盖Mysql的大多数核心服务功能，以及所有的内置函数（如日期、时间、数学和加密函数等），所有跨存储引擎的功能都在这一层实现，比如存储过程、触发器、视图等。

  > 存储引擎

- 负责数据的存储和提取

### 连接器

负责跟客户端建立连接、获取权限、维护和管理连接。

- 连接命令

  ```shell
  mysql -h$ip -P$port -u$user -p
  ```
- 权限
    - 如何获取权限
        - 用户名密码认证通过后，连接器会到权限表里面查出你拥有的权限，之后，这个连接里面的权限判断逻辑，都依赖与此时读到的权限。
- 管理连接
    - 查看连接
      ```shell
      show processlist  
      ```
    - wait_timeout
        - 客户端如果太长时间没动静，连接器就会自动将连接断开
    - 长连接
        - 建立连接的过程通常是比较复杂的
- 查询缓存
    - Mysql拿到一个查询请求后，会先到查询缓存看看，之前是不是执行过这条语句。
        - 查询缓存的失效非常频繁，只要有对一个表的更新，这个表上所有的查询缓存都会被清空。
        - Mysql8.0版本直接将查询缓存的整块功能删掉了。
- 分析器
    - 分析器或先做"词法分析"，再做"语法分析"
        - 词法分析
            - 识别Sql语句里面的字符串分别是什么，代表什么 例如：表名、列名等。
        - 语法分析
            - 根据词法分析的结果，语法分析器会根据语法规则，判断你输入的这个Sql语句是否满足Mysql语法。
- 优化器
    - 经过了分析器，Mysql就知道你要做什么了。在开始执行之前，还要先经过优化器的处理。
        - 索引
            - 优化器是在表里面有多个索引的时候决定使用哪个索引；或者在一个语句有多表关联的时候，决定各个表的连接顺序。
- 执行器
    - Mysql通过分析器知道了你要做什么，通过优化器知道了该怎么做，于是就进入执行器阶段，开始执行语句。
        - 权限判断
            - 开始执行的时候，要先判断一下你读这个表T有没有执行查询的权限。
        - 引擎选择
            - 打开表的时候，执行器就会根据表的引擎定义，去使用这个引擎提供的接口。
        - 例子
            ```mysql
              select * form t where ID = 10;
            ```
        - 执行器的执行流程
            - 调用InnoDB引擎接口去这个表的第一行，判断ID值是不是10，如果不是则跳过，如果是则将这行存在结果集中；
            - 调用引擎接口取"下一行",重复相同的判断逻辑，直接取到这个表的最后一行。
            - 执行器将上述遍历过程中所有满足条件的行组成的记录集作为结果集返回给客户端。

## Mysql日志系统

> Mysql可以恢复到半个月内任意一秒的状态。

### redo log（重做日志）

> 在Mysql里也有这个问题，如果每一次的更新操作都需要写进磁盘，然后磁盘也要找到对应的那条记录，然后再更新，整个过程IO成本、查找成本都很高。

- WAL技术
    - WAL全称Write-Ahead Logging，它的关键点就是先写日志，再写磁盘
- 写入磁盘的时机
    - 当有一条记录需要更新的时候，InnoDB引擎就会先把记录写到redo log里面，并更新内存，这个时候更新就算完成了。同时，InnoDB引擎会在适当的时候，将这个操作记录更新到磁盘里面，而这个更新往往是在系统比较空闲的时候做。
- redo log大小
    - 固定大小，比如可以配置为一组4个文件，每个文件的大小是1GB，从头开始写，写到末尾就又回到开头循环写。
    - write pos是当前记录的位置，一边写一边后移。checkpoint是当前要擦除的位置，也是往后推移并且循环的，擦除记录前要把记录更新到数据文件。
- redo log写满了
    - 先将一部分log写入磁盘，为新的记录腾出空间。
- crash-safe
    - 有了redo log，InnoDB就可以保证即使数据库发生异常重启，之前提交的记录都不会丢失。
    - innodb_flush_log_at_trx_commit这个参数设置成1的时候，表示每次事务的redo log都直接持久化到磁盘。
    - sync_binlog这个参数设置成1的时候，表示每次事务的binlog都持久到磁盘。

### binlog(归档日志)

> server层特有。

- binlog和redo log不同点
    - redo log是InnoDB引擎特有的；binlog是Mysql的是Server层实现的，所有引擎都可以使用。
    - redo log是物理日志，记录的是"在某个数据页上做了什么修改"；binlog是逻辑日志，记录的是这个语句的原始逻辑。
    - redo log是循环写，空间固定会用完；binlog是可以追加写入的。"追加写"是指binlog文件写到一定大小后会切换到下一个，并不会覆盖以前的日志。

- update语句的内部流程
    ```mysql
      update T set N = N + 1 where ID = 2;
    ```
    - 执行器先找引擎取ID=2这一行。ID是主键，引擎直接用树搜索这一行。如果ID=2这一行所在的数据页本来就在内存中，就直接返回给执行器；否则，需要先从磁盘读入内存，然后再返回。
    - 执行器拿到引擎给的行数据，把这个值加上1，比如原来是N，现在就是N+1，得到新的一行数据，再调用引擎接口写入这行数据。
    - 引擎将这行数据更新到内存中，同时将这个更新操作记录到redo log里面，此时redo log处于prepare状态。然后告知执行器执行完成了，随时可以提交事务。
    - 执行器生成这个操作的binlog，并把binlog写入磁盘。
    - 执行器调用引擎的提交事务接口，引擎把刚刚写入的redo log改成commit状态，更新完成

### 执行流程

![avatar](../pics/2e5bff4910ec189fe1ee6e2ecc7b4bbe.png)

### 两阶段提交

> redo log的写入拆成了两个步骤：prepare和commit，这就是两阶段提交。

- 数据恢复
    - 首先，找到最近的一次全量备份，然后从备份的时间点开始，将备份的binlog依次取出，重放到需要恢复的时刻。
- 不使用两阶段提交
    - 数据库的状态就有可能和用它的日志恢复出来的库状态不一致。

<h3 id="3">Mysql事务</h3>
> 事务就是要保证一组数据库操作，要么全部成功，要么全部失败。

- 隔离性与隔离级别
    - ACID 原子性、一致性、隔离性、持久性
    - 多个事务带来的问题 脏读、不可重复读、幻读
    - 隔离级别
      > 隔离级别越高，效率越低  
      Mysql默认的隔离级别是读提交

        - 读未提交：一个事务还未提交时，它做的变更就能被别的事务看到
        - 读提交：一个事务提交之后，它做的变更才会被其他事务看到
        - 可重复读：一个事务执行过程中看到的数据，总是跟这个事务在启动时看到的数据是一致的。当然在可重复读隔离级别下，未提交变更对其他事务也是不可见的。
        - 串行化：顾名思义是对同一行记录，"写"会加"写锁"，"读"会加"读锁"。当出现读写冲突的时候，后访问的事务必须等前一个事务执行完成，才能继续执行。
      



