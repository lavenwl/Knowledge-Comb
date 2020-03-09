### 装饰器模式的定义

\(Decorator Patter, Wrapper Pattern\)也称包装模式,指在不改变原有对象的基础之上, 将功能附加到对象上,提供了比集成更有弹性的替代方案\(扩展原有对象的功能\), 属于结构性模式.

> 适用场景:
>
> 1. 用于扩展一个类的功能或给一个雷添加附加职责
> 2. 动态的给一个对象添加功能,这些功能可以再动态的撤销
>
> 优点:
> 1. 装饰器是继承的有力补充, 比继承灵活, 不改变原有对象的情况下动态的给一个对象扩展功能, 即插即用.
> 2. 通过使用不同的装饰器, 以及这些装饰器类的排列组合, 可以实现不同的效果
> 3. 装饰器完全遵守开闭原则
>
> 缺点:
> 1. 会出现更多类, 增加程序复杂度
> 2. 动态装饰时, 多层装饰会更复杂.

### 装饰器模式在源码中的应用

* JDK中IO相关的类, 如: BufferedReader, InputStream, OutputStream.
* Spring中 TransactionAwareCacheDecorator类
* SpringMVC中HttpHeadResponseDecorator类
* MyBatis中处理缓存的设计org.apache.ibatis.cache.Cache下有decorators目录, 下面放了所有装饰器.如:BlockingCache, FifoCache, LoggingCache, LruCache...

### 装饰器模式与代理模式对比

* 装饰器模式就是代理模式的一种特殊应用, 但是这两种设计模式锁面向的功能扩展面不一样
* 装饰器模式强调自身的功能扩展, Decorator所做的就是增强ConcreteComponent的功能\(也有可能是减弱功能\), 主体对象为ConcreateComponent, 着重类功能的变化.
* 代理模式强调代理过程的控制, Proxy完全掌握对RealSubject的访问控制, 因此, Proxy可以决定对RealSubject进行功能扩展, 



