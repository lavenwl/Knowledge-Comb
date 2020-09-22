### Bean的生命周期(蚂蚁金服)

Spring Bean的完整的生命周期从创建Spring容器开始, 知道最终Spring容器销毁Bean, 其中的关键节点是:

* 实例化`BeanFactoryPostProcessor`实现类

    * 执行`BeanFactoryPostProcessor`的`postProcessBeanFactory()`方法

* 实例化`BeanPostProcessor`实现类

* 实例化`InstantiationAwareBeanPostProcessorAdapter`实现类

    * 执行`InstantiationAwareBeanPostProcessor`的`postProcessBeforeInstantiation()`方法

        > 一下内容为执行Bean构造器, 实例化Bean的过程

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

            > 一下内容为执行Bean的初始化方法

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



