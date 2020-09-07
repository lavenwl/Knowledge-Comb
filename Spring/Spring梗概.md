### Spring的设计初衷

* 基于POJO的轻量级和最小侵入性编程
* 通过依赖注入和面向接口松耦合
* 基于切面和惯性进行声明式编程
* 通过切面模板减少样版式代码

### Spring注解编程的演变

* Spring1.x: Java5刚开始了注解新特性 

    * sprig也支持了 @Transactional注解

    * xml 还是唯一的配置方式

* Spring2.x: 

    * 添加了@Required, @Repository, @Aspect等注解
    * xml提高了扩展能力, 可以自定义标签解析

* Spring2.5: 引入了核心的注解

    * @Autowired 依赖注入
    * @Qualifier 依赖查找
    * @Component, @Service 组件声明
    * @Controller, @RequestMapping等 Spring MVC的注解

    > 虽然提供了很多注解, 但是这个版本还是无法完全脱离xml进行配置, 比如`context:annotation-config`:注册Annotation处理器, `context:component-scan`: 负责扫描classpath下制定的包路径下呗Spring模式注解标注的类,将他们注册成Spring Bean

* Spring3.x: 里程碑版本, 可以完全使用注解配置, 达到去xml化, 使用@Configuraton

    * @Configuration注解实现去xml

    * Enable模块驱动, 自动完成相关组件的Bean的装配

        > Enable模块驱动的原理是: @Enable...注解内有一个@Import注解, @Import注解的参数是传递的一个@Configuration注解的配置类, 在此配置类中加载响应的Bean到IOC容器, 或者加载实现了BeanPostProcessor接口的类去解析自定义的注解来自定义Bean实例化过程.

* Spring4.x: 引入了@Conditional注解实现条件装载bean

* Spring5.x: Indexed: 没用过





### 手写Spring的基本思路

1. 配置阶段
    1. 配置web.xml
    2. 设定init-param
    3. 设定url-pattern
    4. 配置Annotation
2. 初始化阶段
    1. 调用init()方法
    2. IOC容器初始化
    3. 扫描相关的类
    4. 创建实例化并保存至容器(IOC)
    5. 进行DI操作(DI)
    6. 初始化HandlerMapping(MVC)
3. 运行阶段
    1. 调用doPost()/doGet()
    2. 匹配HandlerMapping
    3. 反射调用method.invoke()
    4. response.getWrite().write()



默认单例???作业





1. ApplicationContext
    1. 读取配置文件(BeanDefinition): Properties, xml, yml









### bean的生命周期

参考[生命周期流程图](https://www.cnblogs.com/zrtqsk/p/3735273.html)

[深究Spring中Bean的生命周期](https://www.cnblogs.com/javazhiyin/p/10905294.html)







