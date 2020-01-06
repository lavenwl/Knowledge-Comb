### 事务的定义

事务是数据库管理系统（DBMS）执行过程中的一个逻辑单位，由一个有限的数据库操作序列构成

* 数据库最小工作单位，不可以再分
* 包含一个或多个DML语句（insert, delete, update）（单条DDL（create, drop）和 DCL（grant revoke）也会有事务）

### 事务的四大特性

1. 原子性（Atomicity）：不可再分， 要么成功， 要么失败。在InnoDB中通过undo log来实现的，它记录了数据修改之前的值（逻辑日志），一旦发生异常， 就可以用undo log来实现回滚操作。
2. 一致性（Consistent）：指数据库的完整性约束没有被破坏，事务执行前后都是合法的数据状态，比如主键唯一性，字段长度。除了数据库自身的完整性约束， 还有用户自定义的完整性。
3. 隔离性（Isolation）：多个事务对表或行的并发操作是星湖透明的，互不干扰。
4. 持久性（Durable）：事务一旦提交成功，就是永久性的，不能因为宕机或者重启导致数据恢复到原来的状态。

原子性，隔离性，持久性都是为了实现一致性。

### 持久性怎么实现的

持久性是通过redo log 和double write 双写缓冲来实现的，我们操作数据的时候，会先写道内存的buffer Pool 里面，同事记录redo log，如果刷盘之前出现异常，在重启后就可以读取redo log的内容，写入磁盘保证数据的持久性。

当然，恢复成功的前提是数据页本身没有被破坏，是完整的， 这是通过双写缓冲（double write）保证

### 什么时候出现事务

* 添加@Transaction注解，AOP配置事务后，会使用到事务
* 一条普通的更新语句也会启动事务， 默认开启    
* commit, rollback，或者客户端断开连接会结束一个事务

```sql
show global variables like 'tx_isolation'; -- 查看事务隔离级别
show variables like 'autocommit'; -- 查看自动提交配置
```

### 事务并发带来的问题

1. 脏读：读取到其他事务未提交的数据的情况
2. 不可重复读：读取到其他事务已提交的数据， 导致前后读取到的数据不一致
3. 幻读：一个事务前后读取到的数据不一致，是由于其他事务插入造成的，这种情况叫做幻读

不可重复读与幻读的区别在于：不可重复到是修改或者删除，幻读是插入

### 事务个例级别

> 事务个例级别有一个标准： [SQL92标准](http://www.contrib.andrew.cmu.edu/~shadow/sql/sql1992.txt)

1. 未提交读（Read Uncommitted）： 一个事务可以读取到其他事务未提交的数据，没有解决任何问题
2. 已提交读（Read Committed）：一个事务可以读取到其他事务已提交的数据，不能读取到其他事务未提交的数据，它解决了脏读的问题，但是会出现不可重复读的问题。
3. 可重复度（Repeatable Read）：t这可以解决不可重复读的问题，但是不能解决幻读的问题
4. 串行化（Serializable）：所有事务都串行执行，也就是数据操作需要排队，不存在事务并发的问题，解决了所有的问题。

这是标准，不同的数据库厂商或者存储引擎的实现有差异，如Oracle只实现两种RC（已提交读）和串行化（Serializable）

### InnoDB对事务隔离级别的支持

| 事务隔离级别 | 脏读 | 不可重复读 | 幻读 | InnoDB锁操作 |
| :--- | :--- | :--- | :--- | :--- |
| 未提交度（Read Uncommitted） | 可能 | 可能 | 可能 | 无锁 |
| 已提交读（Read Committed） | 不可能 | 可能 | 可能 | 1.select都是快照读，使用MVCC实现。2.加锁的select都是用记录锁，没有Gap Lock.因此会出现幻读 |
| 可重复度（Repeatable Read） | 不可能 | 不可能 | InnoDB不可能 | 1.select使用快照读（snapshot Read），底层使用MVCC实现，2.加锁的select，update，delete语句使用当前读（current Read）底层使用了记录锁，间隙锁，临键锁 |
| 串行化（Serializable） | 不可能 | 不可能 | 不可能 | 为所有的select语句添加in share mode,会和update,delete互斥 |

InnoDB支持的四个隔离级别与SQL92标准基本一致，隔离级别越高，事务的并发就越低，唯一的区别在于InnoDB在RR级别就解决了幻读的问题，这也是InnoDB默认使用RR作为事务隔离级别的原因，既保证了数据的一致性， 有支持较高的并发度。

### 保证一个事务前后两次读取到的数据结果一致，实现事务个例的量大实现方案

1. 基于锁的并发控制（Lock Based Concurrency Control）（LBCC）：读取数据的时候锁住读取的数据不允许其他事务修改
2. 多版本的并发控制（Multi Version Concurrency Control）\(MVCC\) ：在修改数据的时候为它建立一个副本或快照，后面读取这个快照的内容。

问题：基于锁的并发控制会极大影响操作数据的效率，因为大部分应用读多写少，在一个事务读取时不允许其他事务修改的情况发生概率很高。

MVCC核心思想：可以查询到事务开始之前已经存在的数据，即使它后面被修改或者删除，在当前事务之后新增的数据也是查不到的

### InnoDB如何实现MVCC

InnoDB为每行记录都实现了两个隐藏字段：

1. DB\_TRX\_ID：6字节，插入或更新的最后一个事务的事务ID（理解为创建版本号）
2. DB\_ROLL\_PTR: 7字节，回滚指针（理解为删除版本号）

在InnoDB中，MVCC是通过Undo log实现的。

### MySQL InnoDB锁的基本类型

> 详细介绍参考[官网说明](https://dev.mysql.com/doc/refman/5.7/en/innodb-locking.html)

按照锁的粒度区分：

* 行级锁（Shared and Exclusive Locks）：并发性能高，加锁效率低
* 表级锁（Intention Locks）：并发性能差，加锁效率高

按照锁的算法区分：

* Record Locks
* GapLocks
* Next-Key Locks

按照锁的模式区分（都是行锁）：

* 共享锁（Shared Locks）：也叫读锁，可以重复获取，
* 排它锁（Exclusive Locks）：也叫写锁，一旦某行获取，其他事务不可获取该行的共享锁与排它锁

```sql
-- 添加共享锁
select ... lock in share mode;
-- 排它锁有两种添加方式：1.自动添加 增删改操作；2.手动加锁
select ... for update
```

意向锁（表级别的锁）：

意向锁为数据库自动维护

* 意向共享锁：给一行数据添加共享锁之前，数据库会自动为这张表添加一个意向锁
* 意向排它锁：给一行数据添加排它锁之前，数据库会自动为这张表添加一个排它锁

### 行锁的原理

InnoDB的行锁，是通过锁住索引实现的。

1. 没有索引的表，进行全表扫描，把每一个隐藏的聚集索引都锁住了
2. 二级索引加锁会锁住主键索引，这根辅助索引的执行逻辑有关系

### 锁的算法

1. 记录锁（Record Locks）：当我们使用唯一性索引（包括唯一索引和主键索引）使用等值查询，精准匹配到一条记录，这个时候使用的就是记录锁。它只锁住一行记录。
2. 间隙锁（Gap Locks）：当我们查询的记录不存在，没有命中任何一条记录，无论使用等值查询还是范围查询，他使用的是间隙锁。注意：间隙锁只是阻塞insert，相同间隙锁之间不冲突。
3. 临键锁（Next-Key Locks）：当我们使用了范围查询，不仅仅命中了Record记录，还包含了Gap间隙，这种情况我们使用临键锁。临键锁锁住最后一个key的下一个左开右闭的区间，为什么这样做是为了解决幻读的问题。

### RC与RR的主要区别

1. RR的间隙锁会导致锁定范围扩大。
2. 条件列未使用索引，RR锁表，RC锁行。
3. RC的“半一致性（semi-consistent）”读可以增加update操作的并发性能。

在RC中，一个update语句，如果督导一行已经加锁的记录，此时InnoDB返回记录最近提交的版本，由MySQL上层判断此版本是否满足update的where条件，如果满足（需要更新），则MySQL会重新发起一条读操作，此时会读取最新版本并加锁。

实际上，如果能够正确的使用锁（避免不适用索引去加锁），只锁定需要的数据，用默认的RR级别就可以了。

### 等待锁的事务等待多长时间

MySQL有一个参数用来控制取锁的等待时间，默认50s

```sql
show variables like 'innodb_lock_wait_timeout';
```

### 查看锁的信息

show Status命令中包括了一些行锁的信息

```sql
show status like 'innodb_row_lock_%';
```

| 参数 | 说明 |
| :--- | :--- |
| innodb\_row\_lock\_current\_waits | 当前正在等待锁定的数量 |
| innodb\_row\_lock\_time | 从系统启动到现在锁定的总时间长度，单位ms |
| innodb\_row\_lock\_time\_avg | 每次等待所花平均时间 |
| innodb\_row\_lock\_time\_max | 从系统启动到现在等待时间最长的一次所花的时间 |
| innodb\_row\_lock\_waits | 从系统启动到现在总共等待的次数 |

```sql
--当前运行的所有事务， 还有具体的语句
select * from information_schema.innodb_trx; 

--当前出现的锁
select * from information_schema.innodb_locks;

--锁等待的对应关系
select * from information_schema.innodb_lock_waits;

--如果一个事务长时间持有锁不释放，可以kill 事务对应线程ID，也就是innodb_trx表中的trx_mysql_thread_id
```

### 如何避免死锁

1. 在程序中，操作多张表时，尽量以相同的顺序来访问（避免形成等待环路）
2. 批量操作单张表数据的时候，先对数据进行排序（避免形成等待环路）
3. 申请足够级别的锁，如果操作数据，就是用排它锁
4. 尽量是用索引访问数据，避免没有where条件的操作，避免锁表
5. 如果可以，大事务化成小事务
6. 是用等值查询，而不是范围查询数据，命中记录，避免间隙锁对并发的影响





