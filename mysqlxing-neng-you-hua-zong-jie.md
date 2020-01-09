### MySQL的执行流程

客户端-----------&gt;查询缓存（如果使用了缓存）---&gt;解析器---&gt;预处理器---&gt;查询优化器---&gt;执行计划---&gt;执行引擎---&gt;存储引擎

最后再由执行引擎返回结果到缓存（如果使用了缓存）或客户端。

### 连接优化（配置优化）

1. 服务端：
   1. 修改配置参数增加可用的连接数，修改max\_connections的大小
   2. 及时释放不活动的连接，交互式与非交互式的默认超时时间是28800s，8小时。可以调小这个值
2. 客户端：
   1. 使用连接池，ORM层面（Mybatis）；或者专用的连接池工具（阿里的Druid，Spring Boot2.x版本默认的连接池Hikari）

> 连接池的默认连接数量不是越大越好，推荐为：核数\*2+1，因为连接由线程执行，连接过多会造成CPU过多的上下文切换

### 缓存优化（架构优化）

1. 使用缓存，如:Redis。
2. 主从复制：更新语句会记录binlog，从服务器从主服务器上获取binlog文件，解析里面的SQL语句，在从服务器上执行一遍。
   1. 单线程：主库执行的多条SQL语句，本身存在先后顺序，所以不能并行执行
   2. 异步：主库的写入不关系从库的读写情况
   3. 全同步复制：所有从库全部同步完，主库再返回给客户端。
   4. 半同步复制：等待至少一个从库接收到binlog并写入到relaylog中才返回客户端。使用需要启用插件。**详见SQL**
   5. 多库并行复制：多个从库并行复制不存在并发问题
   6. 异步复制之GTID复制：对主库执行的事务进行分组，并给他们编号，这个编号叫GTID（Global Transaction Identifiers）
3. 分库分表：垂直分库，水平分库

```sql
--启用主从复制的半同步复制（主库执行）
install plugin rpl_semi_sync_master soname 'semisync_master.so';
set global rpl_semi_sync_master_enabled=1;
show variables like '%semi_sync%';

--启用主从复制的半同步复制（从库执行）
install plugin rpl_semi_sync_slave soname 'semisync_slave.so';
set global rpl_semi_sync_slave_enabled=1;
show global variables like '%semi%';

--启动GTID复制,默认关闭
show global variables like 'gtid_mode';
```

### 高可用方案

1. 主从复制：传统的HAProxy + keepalived方案，基于主从复制
2. NDB Cluster: 基于NDB集群存储引擎的MySQL Cluster 详情参见[官网说明](https://dev.mysql.com/doc/mysql-cluster-excerpt/5.7/en/mysql-cluster-overview.html)
3. Galera：一种多主同步复制的集群方案。详情参见[官网说明](https://galeracluster.com)
4. MHA/MMM：Master-Master replication manager for MySQL是一种一主的高可用架构
5. MGR：MySQL5.7.17退出的InnoDB Cluster， 也叫MySQL Group Replication（MGR）

### 优化器优化（SQL语句分析与优化）

#### 慢查询日志 slow query log

##### 打开慢日志开关

```sql
show variables like '%slow_query%';

--也可以直接动态修改参数（重启后失效）
set @@global.slow_query_log=1; -- 1启动，0关闭，重启后失效
set @@global.long_query_time=3; --MySQL默认的慢查询时间是10s
show variables like '%long_query%';
show variables like '%slow_query%';
```

或者修改配置文件my.cnf

```config
slow_query_log=ON
long_query_time=2
slow_query_log_file=/var/lib/mysql/localhost-slow.log
```

##### 慢日志的分析

1. 日志内容
2. mysqldumpslow

> muysqldumpslow工具的详细介绍参见[官网说明](https://dev.mysql.com/doc/refman/5.7/en/mysqldumpslow.html)

```sql
show global status like 'slow_queries'; --查看有多少慢查询
show variables like '%slow_query%'; --获取慢日志的目录

--mysqldumpslow查询用时最多的20条SQL
mysqldumpslow -s t -t 20 -g 'select' /var/lib/lmysql/localhost-slow.log
```

mysqldumpslow查询结果说明

* count：这个SQL执行了多少次
* time：执行的时间，括号里面累计时间
* lock：锁定的时间：括号里面累计时间
* Rows: 返回的记录，括号是累计记录

#### show profile

showprofile可以查看SQL语句执行的时候使用的资源信息，如CPU，IO的消耗情况，在SQL中输入help profile可以看到帮助信息

##### 查看是否开启

```sql
select @@profiling;
set @@profiling=1;
```

##### 查看Profile统计

```sql
--查看统计信息（命令最后带s）
show profiles;
--查看最后一条SQL的执行详细信息从中找出耗时最多的环节（没有s）
show profile;
--也可以根据查询ID查看执行详细信息，在后面加上for query ID.
show profile for query 1;
```

#### 其他系统命令

可以通过查看运行线程状态和服务器运行信息，存储引擎信息来分析

##### show processlist

> 具体介绍详见[官网说明](https://dev.mysql.com/doc/refman/5.7/en/show-processlist.html)

```sql
show processlist;
select * from information_schema.processlist;
```

| 列名 | 说明 |
| :--- | :--- |
| id | 线程的唯一标志，可以根据它kill线程 |
| User | 启动这个线程的用户，普通用户只能看到自己的线程 |
| Host | 哪个IP端口发起的连接 |
| db | 操作的数据库 |
| Command | 线程的命令，详见[官网说明](https://dev.mysql.com/doc/refman/5.7/en/thread-commands.html) |
| Time | 操作持续时间，单位秒 |
| State | 线程状态，详见[官网说明](https://dev.mysql.com/doc/refman/5.7/en/general-thread-states.html) |
| info | SQL语句的前100个字符，如果查看完整SQL使用 show full processlist |

##### show status

show status 用于查看MySQL服务器运行状态（重启后会清空）有session和global两种作用域，详见[官网说明](https://dev.mysql.com/doc/refman/5.7/en/show-engine.html)

```sql
show global status like 'com_select'; -- 查看select次数
```

##### show engine 存储引擎运行信息

show engine用来显示存储引擎的当前运行信息， 包括事务持有的表锁，行锁信息；事务的所等待情况；线程信号量等待；文件IO请求；bufferpool统计信息

```sql
show engine innodb status;

--如果需要将监控信息输出到错误信息文件（15s一次）可以开启输出
show variables like 'innodb_status_output%';
--开启输出
set global innodb_status_output=ON;
set global innodb_status_output_locks=on;

```

#### EXPLAIN 执行计划

> MySQL5.6.3以前只能分析select，之后可以分析update, delete, insert。 x详细信息参见[官网说明](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html)

下面就EXPLAIN查询到的字段做一一介绍：

##### id

* ID是查询的编号
* 


