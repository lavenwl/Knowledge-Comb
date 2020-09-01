# DUBBO发布流程

> 以注解方式进行流程解析
>
> 出注解方式外, dubbo还提供xml方式的解析`dubbo:service` 

注解`@DubboComponentScan`定义了dubbo需要扫描路径, 同时定义加载dubbo的启动类

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import({DubboComponentScanRegistrar.class})
public @interface DubboComponentScan {
    String[] value() default {};

    String[] basePackages() default {};

    Class<?>[] basePackageClasses() default {};
}
```

注解`@Import({DubboComponentScanRegistrar.class})`将`DubboComponentScanRegistrar`类添加到Spring的IOC容器.

> `DubboComponentScanRegistrar`实现了`ImportBeanDefinitionRegistrar`接口,配合@Import注解可以实现Spring动态加载bean的功能.

`DubboComponentScanRegistrar`类的初始化方法, 加载了`ServiceAnnotationBeanPostProcessor`类,

`ServiceAnnotationBeanPostProcessor`类或其父类`ServiceClassPostProcessor`实现了一下几个接口

* `BeanDefinitionRegistryPostProcessor`: 完成spring容器内bean的后置处理

    > 需要重写两个方法
    >
    > * postProcessBeanDefinitionRegistry

* `EnvironmentAware`: 实现此类并重写setEnvironment方法, 则可以获取application.properties文件内的属性

* `ResourceLoaderAware`: 自定义文件加载器

* `BeanClassLoaderAware`: 自定义类加载器

在重写的方法:`postProcessBeanDefinitionRegistry`中注册了一个监听器`DubboBootstrapApplicationListener`

> `DubboBootstrapApplicationListener`继承了抽象类`OneTimeExecutionApplicationContextEventListener`
>
> Spring容器会在上下文装载完成后触发监听方法`onApplicationContextEvent(..)`
>
> 最终的dubbo启动方法就在这里`dubboBootStrap.start()`

在重写的方法:`postProcessBeanDefinitionRegistry`中执行方法`registerServiceBeans()`扫描所有加油Service注解的类, 轮询执行方法`registerServiceBean()`对每一个Service进行注解属性的解析.

### dubboBootStrap.start()

exportService(): 轮询所有的dubbo服务(ServiceConfig)进行发布

> 一个接口发布几次取决于配置了多少中发布协议

执行ServiceConfig的export()方法,  其中准备元数据, 以及执行doExport() 方法

doExport()方法调用doExportUrls()方法

doExportUrls()方法会轮询所有的设置发布协议, 并对每一个协议执行`doExportUrlsFor1Protocol()`对具体的某一个服务的某一种协议进行发布操作

执行具体PROTOCOL.export()方法.

> PROTOCOL是一个适配器, 根据传递的invoke里面的url参数选择不同的注册器的实现类来执行不同的注册逻辑
>
> 这里PROTOCOL对象的获取使用到了dubbo扩展点特性
>
> 最终的PROTOCOL是使用了包装器模式
>
> ProtocolListenerWrapper(QosprotocolWrapper(ProtocolFilterWrapper(dubboprotocol)))
>
> > ProtocolListenerWrapper: 用于export时候插入监听机制
> >
> > QosprotocolWrapper: 如果当前配置了注册中心, 则会启动一个Qos server.qos 是dubbo的在线运维平台, 使用Telnet链接, 默认端口22222
> >
> > ProtocolFilterWrapper: 对invoker进行filter的包装,实现请求过滤

##### 服务注册时的RegistryProtocol.export

* 获取zookeeper注册中心的地址`zookeeper://ip:port`
* 获取服务提供者URL `dubbo://ip:port`
* 订阅重写数据, 当配置发生变化时, 可以通过此订阅通知服务重新暴露服务
* 根据具体的协议暴露服务`doLocalExport()`启动netty server
* 根据invoker中的URL获取registry实例: zookeeperRegistry[扩展点]
* 获取到要注册到注册中心的URL
* 注册到注册中心`register()`

##### 服务发布时的DubboProtocol.export

* openServer: 开启服务
* createServer: 创建服务
    * getTransport()是一个自适应扩展点, 用来对bind方法添加自适应注解, 这里选择通信协议, 如netty, netty4, mina等.
    * NettyTransporter.bind, 返回一个NettyServer
* doOpen: 开启nettyServer

> nettyServer, 在设置Handler时,设置的是一个nettyServerHandler的链路
>
> MultiMessageHandler(heartBeatHandler(AllChannelHandler(DecodeHandler(HeadrExchangeHeandler(DubboProtocol)))))
>
> 后续收到的请求, 会一层一层的处理, 比较繁琐

### 服务发布总结

服务发布过程中几个重要的步骤:

* 构建一个exporterMap, 以服务路径名称为KEY, 把invoker包装成了DubboExporter作为value存储
* 针对同一台机器上的多个服务, 只启动一个服务实例
* 采用Netty4发布服务

### 服务注册流程

沿用上面的服务注册时的RegistryProtocol.export

其中的getRegistry()方法调用的是`registryFactory.getRegistry()`方法

registryFactory对象是通过依赖注入实现的扩展点

最终返回的是通过监听包装过的zookepper注册器ListenerRegistryWrapper(ZookeeperRegistry)

执行ZookeeperRegister方法, 将服务地址注册到注册中心

>  ZookeeperRegistry继承了FailbackRegistry方法, 实现了集群容错的功能, 调用父类的register方法, 使用模板方法, 最终调用了ZookeeperRegistery的doRegister方法