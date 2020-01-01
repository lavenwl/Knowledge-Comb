### 如何查询MySQL链接超时时间

```sql
show global variables like 'wait_timeout'; -- 非交互式超时时间，（JDBC程序）：2800s (8h)
show global variables like 'interactive_timeout'; -- 交互式超时时间，（数据库工具）：2800s (8h)
```

### 如何查看MySQL链接数量

```sql
show global status like 'Thread%';
```

| 链接类型 | 说明 |
| :--- | :--- |
| Threads\_catched | 缓存中的线程连接数 |
| Threads\_connected | 当前打开的连接数 |
| Threads\_created | 处理链接创建的线程数 |
| Threads\_running | 非睡眠状态的连接数，通常指并发连接数 |

### 查看链接的执行状态

```sql
show processlist;
```

> 具体参数信息详见[官网说明](https://dev.mysql.com/doc/refman/5.7/en/show-processlist.html)

| 链接状态 | 说明 |
| :--- | :--- |
| Sleep | 线程正在等待客户端，以向它发送一个新语句 |
| Query | 线程正在执行查询或往客户端发送数据 |
| Locked | 该查询被其他查询锁定 |
| Copying to tmp table on disk | 临时结果集合大于tmp\_table\_size。线程把临时表从存储器内部各式改变为磁盘模式，以节约存储器 |
| Sending data | 线程正在为SELECT语句处理行， 同事正在向客户端发送数据 |
| Sorting for group | 线程正在进行分类， 以满足GROUP BY要求 |
| Sorting for order | 线程正在进行分类，以满足ORDER BY要求 |

> 更多状态才考[官网说明](https://dev.mysql.com/doc/refman/5.7/en/thread-commands.html)

### MySQL最大连接数是多少

```sql
show variables like 'max_connections'; --在5.7版本中默认151个， 最大可设置为16384（2^14）
```

> show 的参数说明：
>
> * 默认级别为Session，全局使用 global级别
> * 动态修改， set 的内容重启后失效， 永久改动修改配置文件  /etc/my.cnf

### MySQL通信协议有哪些

* Unix Socket: 如linux服务器上没有指定-h参数， 则默认使用Socket方式连接
* TCP/IP： 如果添加了-h参数则使用了此种协议连接，这是默认协议， 也是使用最多的协议
* Named Pipes 与 Share Memory 两种协议只使用在Windows环境。

### MySQL使用的通信方式

| 通信方式 | 说明 |
| :--- | :--- |
| 单工 | 数据单向传输 |
| 半双工 | 数据双向传输，但不能同时传输 （MySQL使用的通信方式） |
| 全双工 | 数据双向传输，可以同时传输 |

> 半双工给MySQL带来的问题
>
> * mysql的查询语句不可以过长， 默认4M, 可以对max\_allowed\_\_packet参数进行设置
> * 返回大小受限制，所以sql中尽量避免不含limit的查询。进行分批查询。

### MySQL是否有缓存模块

mysql有自己的缓存模块， 但默认是关闭的。原因如下

* 它要求SQL语句必须一模一样，一个空格也不行
* 表里面任何一条数据发生变化，缓存失效

> mysql的缓存模块在8.0 版本被移除，推荐使用第三方ORM框架， 如：Mybatis；或独立的缓存服务，如：Redis

### MySQL的解析器（Parser）和预处理模块（Preprocessor）做了哪些工作

1. 词法解析：把一个完整的SQL解析成一个个单词，确定数据长度，类型。位置等信息
2. 语法解析：根据SQL语法做解析最终生成**解析树**，如引号括号是否闭合。
3. 预处理：检查**表名或者字段是否存在， **

### 查询优化器（Query Optimizer）与查询执行计划

查询优化器的目的就是根据解析树生成不同的执行计划（Execution Plan），然后选择最优执行计划

> MySQL里面使用的是基于开销（cost）的优化器。具体信息参考[官网说明](https://dev.mysql.com/doc/refman/5.7/en/server-status-variables.html)

```sql
show status like 'Last_query_cost';
```

### 优化器做了什么

简单举例说明：

1. 多张表关联时， 以那张表作为基准表。
2. 多个索引可以使用的时候， 使用哪个索引。

### 优化器如何得到执行计划

1. 打开优化器追踪（默认关闭状态，消耗性能）
2. 执行SQL后查看分析过程
3. 使用完后关闭追踪

```sql
--打开优化器追踪
show variables like 'optimizer_trace';
set optimizer_trace = 'enabled=on'; -- 注意设置的session 与 global级别； 默认session

--查看分析过程
select * from information_schema.optimizer_trace;

--关闭优化器分析过程追踪
set optimizer_trace = 'enabled=off';
show variables like 'optimizer_trace';
```

查询到的结果是json格式：主要分三个阶段：

* SQL准备阶段（join\_preparation）
* SQL优化阶段（join\_optimization）
* SQL执行阶段（join\_execution）

其中 expanded\_query:是最后执行的SQL语句； considered\_execution\_plan:列出了所有的执行计划。

### 如何查看MySQL的执行计划

```sql
explain select * from table; -- 在要执行的SQL前添加 explain 关键字
```

可以查询到多张表关联首先查询哪张表，可能用到哪些索引， 实际用到哪些索引。

> 需要注意：explain 的结果也不一定是最终的执行方式。

### 如何查看MySQL的存储引擎

```sql
show table status from 'gupao';
```

一个数据库可以使用多个存储引擎。

### 如何查看数据库存放数据的路径

```sql
show variables like 'datadir';
```

### 数据库支持的存储引擎

```sql
show engines;
```

> 个存储引擎的特性，详见[官网说明](https://dev.mysql.com/doc/refman/5.7/en/storage-engines.html)

### MyISAM存储引擎的特点

拥有三个物理存储文件

* 支持表级别的锁（插入和更新会锁表）， 不支持事物
* 拥有较高的插入（insert）和查询（select）速度
* 存储了表的行数（count速度更快）
* 只使用于只读类的数据分析项目

> 如何快速插入100万行数据：先使用MyISAM存储引擎插入数据， 再修改存储引擎为InnoDB。

### InnoDB存储引擎的特点

拥有两个物理存储文件

* 支持事物，支持外键。因此数据的完整性，一致性更高
* 支持行级别的锁和表级别的锁。
* 支持读写并发，写不阻塞读（MVCC）。
* 特殊的索引存放方式，可以减少IO， 提升查询效率
* 适合：经常更新的表，存在并发读写或者有事物处理的业务系统。

### Memory存储引擎的特点

拥有一个存储文件

* 把数据存储在内存里面， 读写速度很快， 但是数据库重启或者崩溃， 数据会全部消失，只适合做临时表

### CSV存储引擎的特点

拥有三个文件

* 不允许有空行
* 不支持索引
* 格式通用，可以直接编辑
* 适合：在不同数据库之间导入导出

### 如何选择存储引擎

1. 如果对数据一致性要求比较高，需要事物支持，可以选择InnoDB.
2. 如果数据查询多更新少，对查询性能要求比较高， 可以选择MyISAM。
3. 如果需要一个用于查询的临时表， 可以选择Memory.
4. 如果都不能满足要求， 可以使用C语言自己开发一个存储引擎，详情参考[官网说明](https://dev.mysql.com/doc/internals/en/custom-engine.html)

### MySQL的执行引擎（Query Execution Engine）的作用

直接操作存储引擎， 执行优化器分析后的最优执行计划。

## MySQL体系结构总结

### MySQL模块详解

![](/assets/import.png)

| 模块名称 | 说明 |
| :--- | :--- |
| Connector | 用来支持各种语言和SQL的交互，比如PHP， Python, JDBC |
| Management Serveices & Utilities | 系统管理和控制工具，包括备份恢复，MySQL复制，集群等 |
| Connection Pool | 连接池，管理需要缓存的资源，包括用户密码权限线程等 |
| SQL Interface | 用来接收用户的SQL命令，返回用户需要的查询结果 |
| Parser | 用来解析SQL语句 |
| Optimizer | 查询优化器 |
| Cache and Buffer | 查询缓存，除了行记录的缓存外，还有表缓存，key缓存，权限缓存等 |
| Pluggable Storage Engines | 插件存储引擎， 它提供API给服务层使用 |

 













































