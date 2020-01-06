### 什么是索引

数据库索引，是数据库管理系统（DBMS）中一个排序的数据结构，以协助快速查询、更新数据库表中的数据。

### InnoDB中，索引的类型

在InnoDB中，索引有三种：

* 普通索引（Normal）：也叫非唯一索引，是最普通的索引， 没有任何限制。
* 唯一索引（Unique）：要求键值不能重复，**主键索引**是一种特殊的唯一索引：它要求键值不能为空。
* 全文索引（Fulltext）：针对交大数据如几KB数据，解决like查询效率低的问题。只有文本类型的字段才可以创建全文索引。

### 索引的存储模型推演

> 算法演示工具 [链接](https://www.cs.usfca.edu/~galles/visualization/Algorithms.html)

1. 二分查找：采用数组实现缺点：插入删除效率低；采用链表实现缺点：查找效率低；
2. 二叉查找树（BST Binary Search Tree）：各个子树深浅不一， 造成性能不稳定
3. 平衡二叉树（AVL Tree）:通过左旋右旋操作保证左右子树深度绝对值不能超过1.由于一个节点16K会有浪费IO的情况
4. 多路平衡查找树（B Tree）:分叉数比关键字多1.
5. 加强版多路平衡查找树（B+ Tree）:
   1. 分叉数等于关键字数
   2. 根节点与枝节点都不存储数据，只有叶子节点存储数据， 如果命中也需要查询到对应的叶子节点后取数据。

### InnoDB逻辑存储结构分级

1. 表空间：InnoDB存储引擎逻辑存储结构的最高层，分为：系统表空间，独占表空间，通用表空间，l临时表空间，undo表空间
2. 段（segment）:把空间由段组成，独立表空间（ibd文件）里面会有很多个段组成。创建一个索引会有两个段，一个索引段（non-leaf node segment），一个数据段（leaf node segment）;
3. 簇（extent）:一个段是由多个簇组成，一个簇大小1M\(64个页做成16k\)一个段至少有一个簇，可以有无限多个簇；
4. 页（page）: 页是组成簇的基本单位。一个簇是由连续的（物理和逻辑上）64个页组成。一个表空间最多有2^32个页，64T。
5. 行（row）:InnoDB存储引擎是面向行（Row-oriented）的，行支持两种行格式。

> 行格式的介绍具体参考[官网说明](https://dev.mysql.com/doc/refman/5.7/en/innodb-row-format.html)

### 查看数据和索引的大小

```sql
select CONCAT(ROUND(SUM(DATA_LENGTH/1024/1024),2),'MB') AS data_len, 
CONCAT(ROUND(SUM(INDEX_LENGTH/1024/1024),2),'MB') as index_len 
from information_schema.TABLES 
where table_schema='gupao' 
and table_name='user_innodb';
```

### InnoDB中B+树的特点

1. B 树的变种， 可以解决B树能解决的问题： 更多的路数，存储更多关键字
2. 扫库，扫表能力强：只需要遍历叶子节点， 不需要遍历整个树
3. 更强的磁盘读写能力：根节点与枝节点不存储数据，一次可以加载更多的数据。
4. 排序能力更强：所有的叶子节点形成一个链表
5. 效率更稳定：所有的数据都在叶子节点上，所以IO的次数是稳定的

### 哈希索引的特点

1. 时间复杂度O（1），查询速度快，但不能用于排序。
2. 只支持等值查询， 不支持范围查询。
3. 字段重复值比较多时，会有大量哈希冲突（采用拉链法解决），效率降低。

### InnoDB如何使用哈希索引

InnoDB中为热点数据自动创建自适应Hash索引，用户不可见。这个默认值可以修改：默认打开 

```sql
show variables like 'innodb_adaptive_hash_index';
show engine innodb status; --存储引擎状态中查看
```

### MySQL数据存储文件

数据文件存储的目录：

```sql
show variables like 'datadir';
```

* MyISAM存储引擎：
  * .frm文件是表结构定义文件
  * .MYD文件是数据文件
  * .MYI文件是索引文件
* InnoDB存储引擎：
  * .frm文件是表结构定义文件
  * .ibd文件存放了索引与数据

### 什么是聚集索引（聚簇索引）？

索引键值的逻辑顺序与表数据行的物理顺序一致的索引。

### 如果表中没有主键怎么处理

1. 如果我们定义了主键（Primary key），那么InnoDB会选择主键作为聚集索引。
2. 如果没有显示定义主键，则InnoDB会选择第一个不包含null值的唯一索引作为主键索引。
3. 如果也没有唯一索引，则InnoDB会选择内置6字节长的ROWID作为隐藏的聚集索引，

```sql
select _rowid from tableName;
```

### 索引的使用原则

1. 列的离散度
   1. *  离散度公式：count\(distinct\(column\_name\)\):count\(\*\);
2. 联合索引最左匹配
   1. * 使用及创建联合索引时，最常使用的条件放在前面
      * 联合索引匹配由左到右中间不能中断
3. 覆盖索引
   1. * 回表：非主键索引，我们先通过索引找到主键索引，再通过主键索引找到数据，它基于主键索引的查询多扫描一次索引树， 这个过程叫做索引。
      * 覆盖索引：不管是单列索引还是联合索引， 如果select的数据列只用在索引中就能够取得， 不必从数据区中读取，这时候使用的索引就叫做覆盖索引，这样就避免了回表。
      * EXPLAIN 中 Extra字段含有"Using index"，代表使用了覆盖索引
      * 覆盖索引减少了IO次数，减少了数据的访问量，可以大大提升查询的效率
4. 索引条件下推（ICP）
   1. * 索引的比较实在存储引擎进行的，数据记录的比较是在Server层进行的
      * EXPLAIN 中 Extra字段含有“Using where”, 代表从存储引擎取回的数据不完全满足条件，需要在Server层过滤
      * EXPLAIN 中 Extra字段含有“Using index condition”，代表使用了索引条件下推。
5. 索引的创建与使用
   1. 索引的创建
      1. * 在用于where 判断 order排序 join 的 on字段上创建索引
         * 索引个数不要太多（浪费空间，更新变慢）
         * 区分度第的字段不建立索引（离散度低，导致扫描行数过多）
         * 频繁更新的值， 不要作为主键货色索引（页分裂）
         * 组合索引把散列性高（区分度高）的值放在前面
         * 创建复合索引，而不是修改单列索引
         * 过长的字段，建立前缀索引
         * 不建议使用无序值（UUID，身份证）作为索引
   2. 什么情况应用不到索引
      1. * 索引列上使用函数
         * 字符串不加引号，出现隐形转换
         * like条件前面带“%”
         * 负向查询（not like  !=   not in ）
         * 最终用不用到索引是有优化器说了算，优化器根据cost（Cost Base Optimizer）开销,他不基于规则（Rule-Base Optimizer）也不基于语义，怎么计算开销详见[官网说明](https://dev.mysql.com/doc/refman/5.7/en/cost-model.html)

```sql
alter table tableName drop index indexName; -- 删除一个索引
alter table tablename add index indexName(columnName); -- 添加一个索引
show indexes from tableName; -- 查看所有的索引

-- 索引条件下推
set optimizer_switch='index_condition_pushdown=off'; -- 设置关闭索引条件下推
show variables like 'optimizer_switch'; -- 查看索引条件下推的状态
set optimizer_switch='index_condition_pushdown=on'; -- 开启ICP（索引条件下推）
```





















