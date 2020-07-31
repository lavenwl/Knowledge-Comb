参考[网页](https://www.cnblogs.com/lqh969696/p/10985897.html)

### Vo(value object): 值对象

通常用于业务层之间的数据传递, 由 new 创建, 由GC回收, 和PO一样也是仅仅包含数据而已.但是抽象出业务对象, 可以和表对应,也可以不是.

### PO(Persistant object): 持久层对象

是ORM(Object Relational Mapping)框架中Entity, PO属性和数据库表中的字段形成一一对应的关系, VO和PO, 都是属性加上对应的get和set方法, 表面看没有什么不同, 但是代表的含义完全不同.

### DTO(data transfer object): 数据传输对象

接口之间传递的数据的封装, 如: 表里面有是个字段, 但是页面只需要三个字段. DTO由此产生.

* 一是可以提高数据传输的速度, 
* 二是可以隐藏后端的表结构

### BO(business object): 业务对象

业务逻辑封装的对象, 结合PO或者VO进行业务操作.如购买人是一个VO, 出售人是一个VO, 房子是一个VO, 而这三者组成的一条系统购买记录就是一个BO

### POJO(plian ordinary java object): 简单无规则的JAVA对象

纯传统意义上的Java对象, 最基本的Java bean只有属性加上属性的get和set方法

### DAO(data access object): 数据访问对象

这里需要注意, 是数据访问对象, 不是数据库访问对象, 我理解的是比如在内存的缓存的数据对象 

