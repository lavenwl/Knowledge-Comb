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

InnoDB中为热点数据自动创建自适应Hash索引，用户不可见。这个默认值可以修改：

```sql
show variables like 'innodb_adaptive_hash_index';
show engine innodb status; --存储引擎状态中查看
```



