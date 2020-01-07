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



