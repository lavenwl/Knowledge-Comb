### dubbo 支持的协议类型?

* dubbo
* rmi
* hession
* webservice
* rest
* http

### dubbo调用的几种方式

> 参考[网页](https://developer.aliyun.com/article/608811)

* 同步调用
* 异步调用
* 参数回调
* 事件通知

### dubbo之间的通信方式(本地与远程)

1. 生产者修改配置, 不注册到注册中心
2. 消费者, 修改配置,直接连接生产者IP进行调试.

### springcloud与dubbo的区别

要说两者之间的区别, 我们先来说一下他们之间的背景:

##### 背景:

dubbo: dubbo是阿里巴巴服务化治理的核心框架. 并被广泛应用到旗下各成员站点.

Spring Cloud: 是Spring Source的产物,  其与Pivotal和Netflix是对Java企业界影响力最深入的组织. 而Netflix开源的微服务架构套件就是Spring Cloud 的微服务标准的一种实现. 当然现在dubbo也整合进了Spring Cloud , 成为区别于Netflix的另外一套Spring Cloud微服务标准的一种实现

说完背景, 我们再来说一下社区活跃度:

##### 社区活跃度:

dubbo: 架构基本比较稳定, 代码更新频率较低

SpringCloud: 更新较快. 社区维护比较频繁.

然后我们来说一下框架的完整性:

##### 架构完整度:

|              | Dubbo     | Spring Cloud                 |
| :----------- | :-------- | :--------------------------- |
| 服务注册中心 | Zookeeper | Spring Cloud Netflix Eureka  |
| 服务调用方式 | RPC       | REST API                     |
| 服务网关     | 无        | Spring Cloud Netflix Zuul    |
| 断路器       | 不完善    | Spring Cloud Netflix Hystrix |
| 分布式配置   | 无        | Spring Cloud Config          |
| 服务跟踪     | 无        | Spring Cloud Sleuth          |
| 消息总线     | 无        | Spring Cloud Bus             |
| 数据流       | 无        | Spring Cloud Stream          |
| 批量任务     | 无        | Spring Cloud Task            |
| ……           | ……        | ……                           |

##### RPC & REST

使用RPC有以下痛点, 我们为每个微服务定义了各自的service抽象接口, 使得服务的提供方与服务的调用放存在强依赖. REST接口相对RPC更加轻量化, 不存在代码级别的强依赖.但也会导致接口文档不一致的问题, 这个问题可以使用swagger插件来解决. REST方式的服务依赖比RPC更加灵活.

RPC对平台敏感, 难以简单复用. 当我们提供对外服务时, 都会以REST的方式提供出去, 这样可以实现跨平台的特点,任何一种语言的调用方都可以根据接口定义来实现.

##### 使用

Dubbo实现了服务治理的基础，但是要完成一个完备的微服务架构，还需要在各环节去扩展和完善以保证集群的健康，以减轻开发、测试以及运维各个环节上增加出来的压力，这样才能让各环节人员真正的专注于业务逻辑。而Spring Cloud依然发扬了Spring Source整合一切的作风，以标准化的姿态将一些微服务架构的成熟产品与框架揉为一体，并继承了Spring Boot简单配置、快速开发、轻松部署的特点，让原本复杂的架构工作变得相对容易上手一些（如果您读过我之前关于Spring Cloud的一些核心组件使用的文章，应该能体会这些让人兴奋而激动的特性，传送门）。所以，如果选择Dubbo请务必在各个环节做好整套解决方案的准备，不然很可能随着服务数量的增长，整个团队都将疲于应付各种架构上不足引起的困难。而如果选择Spring Cloud，相对来说每个环节都已经有了对应的组件支持，可能有些也不一定能满足你所有的需求，但是其活跃的社区与高速的迭代进度也会是你可以依靠的强大后盾。

##### 文档

dubbo使用的是中文英文都很健全, 由于版本基本确定, 文档间的差异也不多.

SpringCloud 由于版本迭代较快, 存在版本说明差异.

总结来说, dubbo像是组装电脑, 而springCloud 则像是在买品牌机.

### dubbo与spring集成

1. 创建一个接口项目

2. 创建一个实体项目, 引入dubbo的jar包

3. 在application.xml中添加dubbo的相关配置

    > 如: 服务的名称, 服务的注册中心, 服务的url,等信息

### dubbo中的设计模式及例子

* 责任链模式

    责任链模式在dubbo中发挥着举足轻重的作用, 就像是dubbo的骨架. dubbo的调用链组织是用责任链模式串联起来的, 责任链中每一个节点实现Filter接口, 然后由ProtocolFilterWrapper, 将所有的Filter串连起来. dubbo的很多功能都是Filter扩展实现的, 不如监控, 日志, 缓存安全, telnet以及RPC本身都是. 这种模式易于dubbo的扩展.

* 观察者模式 (Listener)

    Dubbo中使用观察者模式最典型的例子是`RegistryService`。消费者在初始化的时候回调用subscribe方法，注册一个观察者，如果观察者引用的服务地址列表发生改变，就会通过`NotifyListener`通知消费者。此外，Dubbo的`InvokerListener`、`ExporterListener` 也实现了观察者模式，只要实现该接口，并注册，就可以接收到consumer端调用refer和provider端调用export的通知。Dubbo的注册/订阅模型和观察者模式就是天生一对。

* 装饰器模式 (Wrapper)

    Dubbo中还大量用到了修饰器模式。比如`ProtocolFilterWrapper`类是对Protocol类的修饰。在export和refer方法中，配合责任链模式，把Filter组装成责任链，实现对Protocol功能的修饰。其他还有`ProtocolListenerWrapper`、 `ListenerInvokerWrapper`、`InvokerWrapper`等。个人感觉，修饰器模式是一把双刃剑，一方面用它可以方便地扩展类的功能，而且对用户无感，但另一方面，过多地使用修饰器模式不利于理解，因为一个类可能经过层层修饰，最终的行为已经和原始行为偏离较大。

* 工厂方法

    `CacheFactory`的实现采用的是工厂方法模式。`CacheFactory`接口定义getCache方法，然后定义一个`AbstractCacheFactory`抽象类实现`CacheFactory`，并将实际创建cache的createCache方法分离出来，并设置为抽象方法。这样具体cache的创建工作就留给具体的子类去完成。

* 适配器模式

    为了让用户根据自己的需求选择日志组件，Dubbo自定义了自己的Logger接口，并为常见的日志组件（包括jcl, jdk, log4j, slf4j）提供相应的适配器。并且利用简单工厂模式提供一个`LoggerFactory`，客户可以创建抽象的Dubbo自定义`Logger`，而无需关心实际使用的日志组件类型。在LoggerFactory初始化时，客户通过设置系统变量的方式选择自己所用的日志组件，这样提供了很大的灵活性。

* 代理模式

    Dubbo consumer使用`Proxy`类创建远程服务的本地代理，本地代理实现和远程服务一样的接口，并且屏蔽了网络通信的细节，使得用户在使用本地代理的时候，感觉和使用本地服务一样。

    