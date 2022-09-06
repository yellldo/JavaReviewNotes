## Mysql日志

> 分为：redo log、undo log、binlog、error log、relay log、slow query log、general log

### redo log（重做日志）

> 确保事务的持久性。redo日志记录事务执行后的状态，用来恢复未写入data file的已成功事务更新的数据。防止在发生故障的时间点，尚有脏页未写入磁盘，在重启mysql服务的时候，根据redo log进行重做，从而达到事务的持久性这一特性。

- 什么时候产生

> 事务开始之后就产生redo log，redo log的落盘并不是随着事务的提交才写入的，而是在事务的执行过程中，便开始写入redo log文件中。

- 什么时候释放

> 当对应事务的脏页写入到磁盘之后，redo log的使命也就完成了，重做日志占用的空间就可以重用（被覆盖）。

### undo log（回滚日志）

> undo log有两个作用：提供回滚和多个版本控制(MVCC)  
> undo log是逻辑日志，

- 什么时候产生
  > 事务开始之前，将当前是的版本生成undo log，undo 也会产生 redo 来保证undo log的可靠性
- 什么时候释放
  > 当事务提交之后，undo log并不能立马被删除，而是放入待清理的链表，由purge线程判断是否由其他事务在使用undo段中表的上一个事务之前的版本信息，决定是否可以清理undo log的日志空间。

### binlog （二进制日志）

> 用于复制，在主从复制中，从库利用主库上的binlog进行重播，实现主从同步。  
> 用于数据库的基于时间点的还原。

- 什么时候产生

> 事务提交的时候，一次性将事务中的sql语句（一个事物可能对应多个sql语句）按照一定的格式记录到binlog中。

- 什么时候释放

> binlog的默认是保持时间由参数expire_logs_days配置，也就是说对于非活动的日志文件，在生成时间超过expire_logs_days配置的天数之后，会被自动删除。

### relay log（中继日志）

> 主从复制的核心就是relay log

### slow query log（慢查询日志）

> 慢日志记录执行时间过长和没有使用索引的查询语句，报错select、update、delete以及insert语句，慢日志只会记录执行成功的语句。

### general log（普通查询日志）

> 记录了服务器接收到的每一个查询或是命令，无论这些查询或是命令是否正确甚至是否包含语法错误，general log 都会将其记录下来 ，记录的格式为 {Time ，Id ，Command，Argument }。也正因为mysql服务器需要不断地记录日志，开启General log会产生不小的系统开销。 因此，Mysql默认是把General log关闭的。