### Bean的生命周期(蚂蚁金服)

Spring Bean的完整的生命周期从创建Spring容器开始, 直到最终Spring容器销毁Bean, 其中的关键节点是:

* 实例化`BeanFactoryPostProcessor`实现类

    * 执行`BeanFactoryPostProcessor`的`postProcessBeanFactory()`方法

* 实例化`BeanPostProcessor`实现类

* 实例化`InstantiationAwareBeanPostProcessorAdapter`实现类

    * 执行`InstantiationAwareBeanPostProcessor`的`postProcessBeforeInstantiation()`方法

        > 以下内容为执行Bean构造器, 实例化Bean的过程

        * 执行`InstantiationAwareBeanPostProcessor`的`postProcessPropertiesValues()`方法

            > 为Bean注入自定义属性

        * **调用`BeanNameAware.setBeanName()`**

        * 调用`BeanClassLoaderAware.setBeanClassLoader()`

        * **调用`BeanFactoryAware.setBeanFactory()`**

        * 调用`EnvironmentAware.setEnviroment()`

        * `EmbeddedValueResolverAware.setEmbeddedValueResolver()`

        * `ResourceLoaderAware.setResourceLoader()`

        * `ApplicationEventPublisherAware.setApplicationEventpublisher()`

        * `MessageSourceAware.setMessageSource()`

        * **调用`ApplicationContextAware.setApplicationContext()`**

        * 如果是webApplication: `ServletContextAware.setServeletContext()`

        * 执行`BeanPostProcessor`的`postProcessBeforeInitialization()`方法

            > 以下内容为执行Bean的初始化方法

            * 调用`InitializingBean`的`afterPropertiesSet()`方法
            * 调用<bean>的`init-method`属性指定的初始化方法

        * 执行`BeanPostProcessor`的`postProcessAfterInitialization()`方法

    * 执行`InstantiationAwareBeanPostProcessor`的`postProcessAfterInstantiation()`方法

> 到此为止, 容器初始化完成, 可以执行正常调用, 下面是销毁的流程

* 调用`DisposibaleBean`的`destory()`方法
* 调用<bean>的`destory-method`属性指定的销毁方法

> Bean的完整声明周期经理了各种方法调用, 这些方法可以分为一下几个层级
>
> 1. Bean自身的方法: 包括Bean本身调用的方法, 和通过配置文件中的<bean>配置的init-method, destory-method指定的方法
> 2. Bean级别生命周期接口方法: 包括BeanNameAware, BeanFactoryAware, InitializingBean, DisposableBean这些接口方法
> 3. 容器级别生命周期接口方法: 包括InstantiationAwareBeanPostProcessor, BeanPostProcessor
> 4. 工厂后处理器接口方法: 包括AspectJWeavingEnabler, ConfigurationClassPostProcessor, CustomAutowireConfigurer等非常有用的工厂后处理器, 工厂后处理器也是容器级别的, 在应用上下文装配配置文件后立即调用.

### springboot的SPI机制

在Springboot自动装配的过程中, 会加载META-INF/spring.factories文件, 加载的过程是由SPringFactoriesLoader加载的. 从Classpath下每个jar包中搜索所有的META-INF/spring.factories文件, 根据配置的路径将对应的类添加到IOC容器中.

### spring AOP 的原理

在日常开发中, 如参数检查, 日志, 异常处理等逻辑在所有的请求中都会涉及, 为了将相应的功能抽离出来, 在所有的功能中提供了一种面向切面的抽离式编程方式, 叫做AOP. 他的实现原理是以代理的方式来实现相应功能的增强.

### springAOP原理源码级别如何实现

//TODO

### Spring中的默认事物, 及事物的集中备选, 各自有什么区别, 默认的事物传递是什么

| 常量名称                  | 常量解释                                                     |
| ------------------------- | ------------------------------------------------------------ |
| PROPAGATION_REQURED       | 支持当前事务, 如果当前没有事务, 就新建一个事务, 这是最常见的事务, 也是Spring默认的事务传播 |
| PROPAGATION_REQUIRES_NEW  | 新建事务, 如果当前存在事务, 把当前事务挂起,新建一个事务, 两个事务相互独立, 外部事务失败回滚, 不影响内部事务, 内部事务失败抛出异常, 外部事务捕获, 也可以不处理回滚操作 |
| PROPAGATION_SUPPORTS      | 支持当前事务, 如果没有事务就以非事务方式执行                 |
| PROPAGATION_MANDATORY     | 支持当前事务, 如果没有事务, 就抛出异常                       |
| PROPAGATION_NOT_SUPPORTED | 以非事务方式操作, 如果当前存在事务, 就把当前事务挂起.        |
| PROPAGATION_NEVER         | 以非事务方式执行, 如果存在事务, 则抛出异常                   |
| PROPAGATION_NESTED        | 如果一个活动的事务存在, 则运行在一个嵌套的事务中, 如果没有活动的事务, 就按照required属性, 他使用一个单独的事务, 这个事务拥有多个回滚保存点, 内部事务回滚不会对外部事务产生影响, 它只对DataSourceTransactionManager事务管理器起效. |

### spring的对象是否是单例的，如果要多例怎么弄，存在线程安全问题吗？如何防止。

spring默认是单例的, 也可以设置为多利, 只要在定义bean的地方设置scope为prototype就可以了.

spring默认使用单例模式, 就会造成controller中的实例变量在多线程访问时出现线程安全的问题, 解决方法是可以使用ThreadLocal的方式获取存储对应的变量值, 也可以设置对应的单例模式为原型模式.



### bean有多少种scope

>  参考https://blog.csdn.net/xiaoyiaoyou/article/details/45876073

初始状态有两种scope:

1. singleton
2. prototype

后续添加了三种状态, 之应用于web应用中

1. request
2. session
3. global session

### spring中的设计模式

// TODO

### @Transaction注解在什么情况下会失效

> 这个是真的学到了

当一个类下有两个方法a(),b()时, 如果b()方法添加了@Transaction注解, 如果在方法a()中调用了方法b(), 则会出现事务失效的情况. 为什么?

因为spring中的事务时通过切面的方式实现的, 在spring加载这个类的时候会生成一个动态代理类, 其中事务的逻辑会织入到对应的方法中, 但是类中方法相互调用使用的是this指针的形式来调用的, 所以, a()方法中调用的b()方法使用的是this.b(), 所以没有使用到代理类中的事务代码. 如何解决这个问题?

使用AOPUtils.getBean().方法名()来解决这个问题.

### Spring的依赖注入方式有哪几种各有什么优缺点

> ?????

### Spring如何解决循环依赖, 在解决循环依赖中需要注意什么

### beanfactoryPostProcessor 作用 beanPostProcessor作用

### spring 的bean的加载过程, 及个组件的作用(修改一个配置文件中的数值) 背!!

### 不同环境下类加载不同的类的问题

### transactional注解，异常能否外围捕获

### spring类加载过程

### spring事物传播的应用log报错(如何具体使用)

### 说一下ApplicationContext的加载过程?

1. https://blog.csdn.net/weixin_42440637/article/details/112074442

[一篇文章搞懂Spring单例循环依赖加载](https://blog.csdn.net/weixin_42440637/article/details/112254141)

[为什么Spring要使用三级缓存解决循环依赖？](https://blog.csdn.net/weixin_42440637/article/details/112264648)

