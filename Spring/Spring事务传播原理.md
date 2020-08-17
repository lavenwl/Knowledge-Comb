### Spring 事务

spring事务时基于AOP实现的

### Spring事务的基础配置

```xml
<bean name="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource"></property>
</bean>
<tx:advice id="txAdvice" transaction-manager="transactionManager">
  <tx:attributes>
    <tx:method name="delete*" propagation="REQUIRED" read-only="false"
               rollback-for="java.lang.Exception" />
    <tx:method name="insert*" propagation="REQUIRED" read-only="false"
               rollback-for="java.lang.Exception" />
    <tx:method name="update*" propagation="REQUIRED" read-only="false"
               rollback-for="java.lang.Exception" />
    <tx:method name="save*" propagation="REQUIRED" read-only="false"
               rollback-for="java.lang.Exception" />
    <tx:method name="edit*" propagation="REQUIRED" read-only="false"
               rollback-for="java.lang.Exception" />
    <tx:method name="apply*" propagation="REQUIRED" read-only="false"
               rollback-for="java.lang.Exception" />
  </tx:attributes>
</tx:advice>

<aop:aspectj-autoproxy proxy-target-class="true" />

<!-- 事物处理 -->
<aop:config>
  <aop:pointcut id="pc" expression="execution(* com.nh.ifarm.*.service..*Impl.*(..))" />
  <aop:advisor pointcut-ref="pc" advice-ref="txAdvice" />
</aop:config>

```

### 事务的四个属性(ACID)

Atomic(原子性):强调操作的不可分割性, 基本工作单位, 要么一起成功, 要么一起失败

Consistency(一致性) :是指事务的前后是从一种一致性状态, 转变为另外一种一致性, 比如转账前AB账户的前是一个数, 转账后,AB两个账户的总数还是这个数.

Isolate(隔离性) :强调多个事务之间不能相互影响.比如A转了两百, 花了一百, 如果两个事务同时发生, 不能出现账户余额不是一百的情况.

Durability(持久性): 指一个事务一但提交, 则永久生效. 不会因为设备故障, 或断点的外部因素影响.

### Spring 事务传播属性

所谓事务传播属性, 是指存在多个事务同时存在的时候, spring应该如何处理这些事务的行为

> propergation: 传播

| 常量名称                  | 常量解释                                                     |
| ------------------------- | ------------------------------------------------------------ |
| PROPAGATION_REQURED       | 支持当前事务, 如果当前没有事务, 就新建一个事务, 这是最常见的事务, 也是Spring默认的事务传播 |
| PROPAGATION_REQUIRES_NEW  | 新建事务, 如果当前存在事务, 把当前事务挂起,新建一个事务, 两个事务相互独立, 外部事务失败回滚, 不影响内部事务, 内部事务失败抛出异常, 外部事务捕获, 也可以不处理回滚操作 |
| PROPAGATION_SUPPORTS      | 支持当前事务, 如果没有事务就以非事务方式执行                 |
| PROPAGATION_MANDATORY     | 支持当前事务, 如果没有事务, 就抛出异常                       |
| PROPAGATION_NOT_SUPPORTED | 以非事务方式操作, 如果当前存在事务, 就把当前事务挂起.        |
| PROPAGATION_NEVER         | 以非事务方式执行, 如果存在事务, 则抛出异常                   |
| PROPAGATION_NESTED        | 如果一个活动的事务存在, 则运行在一个嵌套的事务中, 如果没有活动的事务, 就按照required属性, 他使用一个单独的事务, 这个事务拥有多个回滚保存点, 内部事务回滚不会对外部事务产生影响, 它只对DataSourceTransactionManager事务管理器起效. |

### 事务导致的三大问题

* 脏读:一个事务可以读取到另外一个事务未提交的数据 
* 不可重复读:一个事务中的两次读操作读到了不同的数据是因为另外一个事务的提交造成的
* 幻读:事务一对一定范围的数据进行批量修改, 而另外一个事务插入一条心的数据并提交, 从而导致第一个事务无法修改新增数据, 这种由于一个事务新增导致其他事务不正常的情况称为幻读.

### 数据库事务隔离级别

| 隔离级别         | 隔离级别的值 | 导致的问题                           |
| ---------------- | ------------ | ------------------------------------ |
| Read-Uncommitted | 0            | 导致脏读                             |
| Read-Committed   | 1            | 避免脏读, 导致不可重复读             |
| Repeatable-Read  | 2            | 避免不可重复度, 不可重复读, 导致幻读 |
| Serializable     | 3            | 避免三种问题,但是效率慢              |

### Spring中的事务隔离级别

| 事务隔离级别               | 解释               |
| -------------------------- | ------------------ |
| ISOLATION_DEFAULT          | 使用默认的隔离级别 |
| ISOLATION_READ_UNCOMMITTED | 同上               |
| ISOLATION_READ_COMMITTED   | 同上               |
| ISOLATION_REPEATABLE_READ  | 同上               |
| ISOLATION_SERIALIZABLE     | 同上               |

