参考:

 [【课堂笔记】Redis分布式篇.pdf](source/[课堂笔记]Redis分布式篇.pdf) 

 [【课堂笔记】Redis基础篇.pdf](source/[课堂笔记]Redis基础篇.pdf) 

 [【课堂笔记】Redis实战篇.pdf](source/[课堂笔记]Redis实战篇.pdf) 

 [【课堂笔记】Redis原理篇.pdf](source/[课堂笔记]Redis原理篇.pdf) 

### Redis的启动:

```
redis-server: 启动服务端
redis-cli: 启动客户端连接服务端
```

### 常用命令

```
命令	用途
set key value	设置 key 的值
get key	获取 key 的值
exists key	查看此 key 是否存在
keys *	查看所有的 key
flushall	消除所有的 key
```



缓存问题及解决办法

1. 缓存穿透: 
   1. 定义: 如果缓存中没有对应的数据, 导致大量查询到达数据库, 承载过大问题.
   2. 方案: 加锁以控制数据库的数量
2. 缓存击穿: 
   1. 定义: 缓存中没有数据, 查询数据库后也没有数据, 导致的重复查询数据库的问题.
   2. 方案: 缓存null数据避免多次查询, 布隆过滤器
3. 缓存雪崩: 
   1. 定义: 缓存的大量数据在同一时间过期, 导致集中查询数据的的问题
   2. 方案:  添加随机过期时间

