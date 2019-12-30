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
> * mysql的查询语句不可以过长， 默认4M, 可以对max_allowed\__packet参数进行设置
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



















