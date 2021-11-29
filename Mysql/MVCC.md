## InnoDB对MVCC的实现

`MVCC`的实现依赖**隐藏字段、undo log、Read View**。在内部实现中，`InnoDB`通过数据行的`DB_TRX_ID`和`Read View`来判断数据的可见性，如不可见，则通过数据行的`DB_ROLL_PTR`
找到`undo log`中的历史版本。每个事务读到的数据版本可能不一样，在同一个事务中，用户只能看到该事务创建的`Read View`之前已经提交的修改和该事务本身做的修改。

### 隐藏字段

在内部`InnoDB`存储引擎为每行数据添加三个隐藏字段

- `DB_TRX_ID(6字节)`:表示最后一次插入或更新的事务id。此外,`delete`操作在内部视为更新,只不过会在记录头`Record header`中的`deleted_flag`字段将其标记为已删除
- `DB_ROLL_PTR(7字节)`:回滚指针，指向该行的`undo log`。如果该行未被更新，则为空。
- `DB_ROW_ID(6字节)`:如果没有主键且该表没有唯一非空索引时，`InnoDB`会使用该id作为聚簇索引

### Read View

```c
class ReadView {
 private:
    trx_id_t m_low_limit_id;    /*大于等于这个ID的事务均不可见*/
    trx_id_t m_up_limit_id;     /*小于这个ID的事务均可见*/
    trx_id_t m_creator_trx_id;  /*创建该Read View的事务ID*/
    trx_id_t m_low_limit_no;    /*事务Number，小于该Number的undo logs均可以被Purge*/
    ids_t m_ids;                /*创建Read View时的活跃事务列表*/ 
    m_closed;                   /*标记Read View是否close*/
  
}
```

`Read View`主要是用来做可见性判断，里面保存了当前对事务不可见的其他活跃事务。 主要有以下字段：

- `m_low_limit_id`:目前出现过最大的事务ID+1，即下一个将被分配的事务ID。大于等于这个ID的数据版本不可见。
- `m_up_limit_id`:活跃事务列表`m_ids`中最小的事务ID，如果`m_ids`为空，则`m_up_limit_id`为`m_low_limit_id`。小于这个ID的数据版本均可见。
- `m_ids`:`Read View`创建时其他未提交的活跃事务ID列表。创建`Read View`时，将当前未提交事务ID记录下来，后续即使它们修改了记录行的值，对于当前事务也是不可见的。`m_ids`
  不包括当前事务自己和已提交的事务（正在内存中）。
- `m_creator_trx_id`:创建该`Read View`的事务ID。

### undo log

`undo log`主要有两个作用：

- 当事务回滚时用于将数据恢复到修改之前的样子。
- 另外一个作用是`MVCC`，当读记录时，若该事务被其他事务占用或当前版本对该事务不可见，则可通过`undo log`读取之前的版本数据，以此实现非锁定读。

## **在InnoDB存储引擎中`undo log`分为两种:`insert undo log`和 `update undo log`**:

- `insert undo log`:指在`insert`操作中产生的`undo log`，因为`insert`操作的记录只对事务本身可见，对其他事务不可见，故该`undo log`可以在事务提交后删除。不需要进行`purge`操作
- `update undo log`:`update`或`delete`操作中产生的`undo log`。该`undo log`可能需要提供`MVCC`机制，因此不能在事务提交时就进行删除。提交时放入`undo log`
  链表，等待`purge线程`进行最后的删除

### 数据可见性

在`InnoDB`存储引擎中，创建一个新事务后，执行每个`select`语句前，都会创建一个快照`Redd View`，**快照中保存了当前数据库系统中正在处于活跃（没有Commit的）的事务ID号**
。其实简单的说保存的是系统中当前不应该被本事务看到的其他事务ID列表（即m_ids）。当用户在这个事务中要读取某个记录行的时候，`InnoDB`会将该记录的`DB_TRX_ID`与`Read View`中的一些变量以当前事务ID进行比较，
判断是否满足可见性条件。

### RC和RR隔离级别下MVCC的差异

在事务隔离级别`RC`和`RR`（InnoDB存储引擎的默认事务隔离级别）下，`InnoDB`存储引擎使用`MVCC`（非锁定一致性读），但它们生成`Read View`的时机却不同

- 在RC隔离级别下的`每次select`查询前都生成一个`Read View`（m_ids列表）
- 在RR隔离级别下只在事务开始后`每一此select`数据前生成一个`Read View`（m_ids列表）

### MVCC解决不可重复的问题

虽然RC和RR都通过`MVCC`来读取快照读，但是由于**生成Read View时机不同**，从而在RR级别下实现可重复读。