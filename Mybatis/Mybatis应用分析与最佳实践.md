> 参考文档:  [【课堂笔记】MyBatis应用分析与最佳实践.pdf](../../../../Downloads/[课堂笔记]MyBatis应用分析与最佳实践.pdf) 

### 如何简化JDBC

* Apache 在2003年发布Commons DbUtils工具类, QueryRunner类提供对数据库增删改查的封装.
* Spring JDBC 是由Spring对JDBC进行的封装. JDBCTemplate类封装了各种各样的execute, query, update 方法, 是线程安全的. 初始化可以设置数据源, 解决资源管理问题.
* Hibernate ORM 框架
* MyBatis框架

### Mybatis核心对象的生命周期

1. SqlSessionFactoryBuilder: 用来构建SqlSessionFactory, 而SqlSessionFactory只需要一个, 所以构建一个SqlSessionFactory, 它的使命就完成了, 也就没有存在的意义, 所以它的声明周期只存在于`方法的局部`.
2. SqlSessionFactory(单例): 用来创建SqlSession的, 每次应用访问数据库都需要创建一个会话, 因为我们一直有创建会话的需要, 所以SqlSessionFactory应该存在于应用的整个生命周期中(`作用域是应用的作用域`)
3. SqlSession是一个会话, 因为它不是线程安全的, 不能在线程间共享. 所以我们在请求开始的时候创建一个SqlSession对象, 在请求结束或者方法执行完毕的时候要及时关闭它(`一次请求或者操作中`)
4. Mapper(实际上是一个代理对象) 是从SqlSession中获取的

### 核心配置解读

1. configuration 是整个配置文件的根标签, 实际上对应着MyBatis里面最重要的配置类Configuration, 它贯穿MyBatis执行流程的每一个环节

2. properties 用来配置参数信息, 比如最常用的数据库链接信息. 为了避免直接把参数写死在xml配置文件中, 我们可以吧这些参数单独放在properties文件中, 用properties标签引入进来, 然后在xml配置文件中用`${}`引用就可以了, 可以用resource引用应用里面的相对路径,

3. settings 里面是MyBatis的一些核心配置

4. typeAliases 是类型的别名, 跟linux系统里面的alias一样, 主要用来简化类名全路径的拼写. 比如我们的参数类型和返回值类型都可能会用到我们的Bean, 如果每个地方都配置全路径的话, 那么内容就比较多, 还可能会写错. 我们可以为自己的Bean创建别名, 既可以指定单个类, 也可以指定一个package, 自动转换.

    ```xml
    <typeAliases>
    	<typeAlias alias="blog" type="com.gupaoedu.domain.Blog" />
    </typeAliases>
    <!--配置了别名可以直接在配置文件中使用-->
    <select id="selectBlogByBean" parameterType="blog" resultType="blog">
      select * from blog
    </select>
    ```

    > MyBatis里面有很多系统预先定义好的类型别名, 在TypeAliasRegistry中, 所以可以用string 代替java.lang.String

5. typeHandlers 对数据库值与Java对象的值进行互相转换的对象. MyBatis已经内置了很多TypeHandler(在type包下), 它们全部注册在TypeHandlerRegistry中, 它们都继承了抽象类BaseTypeHandler, 泛型就是要处理的Java数据类型

    > 如果我们需要自定义一些类型转换规则, 或者要再处理类型的时候做一些特殊的动作, 就可以编写自己的TypeHandler, 跟系统自定义的TypeHandler一样, 继承抽象类BaseTypeHandler<T>. 有4个抽象方法必须实现, 
    >
    > | 从Java类型到JDBC类型              | 从JDBC类型到JAVA类型                                     |
    > | --------------------------------- | -------------------------------------------------------- |
    > | setNonNullParameter: 设置非空参数 | getNullableResult:获取空结果集(根据列名, 一般都调用这个) |
    > |                                   | getNullableResult: 获取空结果集(根据下标值)              |
    > |                                   | getNullableResult: 存储过程用的                          |

6. objectFactory 专门用来创建对象的实例, ObjectFactory有一个默认的实现类DefaultObjectFactory, 创建对象的方法最终都是调用了instantiateClass(), 这里面能看到反射的代码.默认情况下, 所有的对象都是由DefaultObjectFactory创建的.

    > 如果想要修改对象工厂在初始化实例类的时候的行为, 就可以通过创建自己的对象工厂, 继承DefaultObjectFactory来实现(不再需要实现ObjectFactory接口).
    >
    > 如果在config文件里面注册, 在创建对象的时候就会被自动调用
    >
    > ```xml
    > <objectFactory type="org.mybatis.example.GPObjectFactory">
    > 	<property name="gupao" value="234"></property>
    > </objectFactory>
    > ```

7. plugins 插件 Mybatis预留了插件的接口, 让Mybatis更容易扩展

    | 类(或接口)        | 方法                                                         |
    | ----------------- | ------------------------------------------------------------ |
    | Executor          | update, query, flushStatements, commit, rollback, getTransaction, close, isClosed |
    | ParameteraHandler | getParamenterObject, setParameters                           |
    | ResultSetHandler  | handlerResultSets, handleOutputParameters                    |
    | StatementHandler  | prepare, parameterize, batch, update, query                  |

8. environments, environment 用来管理数据库的环境, 不同的环境使用不同的数据源

    * transactionManager:
        * 如果配置的JDBC, 则会使用Connection对象的commit(),rollback(),close()管理事务.
        * 如果配置成MANAGED, 会把事务交给容器来管理, 比如JBOSS, Weblogic
        * 如果是Spring+Mybatis, 则没有必要配置, 因为我们会直接在applicationContext.xml里面配置数据源和事务, 覆盖Mybatis的配置.
    * dataSource(数据源):
        * 数据的来源, 在JAVA里面他是对数据库链接的一个抽象
        * 一般数据源都会包括连接池管理的功能, 很多时候会把DataSource直接成为连接池. 准确的说法是: 带有连接池功能的数据源.
        * Mybatis自带两种数据源, UNPOOLED, POOLED, 也可以配置其他的数据源, 如C3P0, Hikari等等, 
        * 再跟Spring继承的时候, 事务和数据源都交给Spring来管理, 不再使用MyBatis配置的数据源.

9. mappers

10. settings

11. 