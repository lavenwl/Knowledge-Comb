BeanNameAware: 获取容器中Bean的名称

> 需要重写setBeanName(BeanName beanName)方法

BeanFactoryAware: 获取当前BeanFactory, 这样可以调用容器的服务

> 重写setBeanFactory(BeanFactory beanFactory)方法

ApplicationContextAware: 获取当前的ApplicationContextAware, 这样可以调用容器的服务

> 重写setApplicationContext(ApplicationContext applicationContext)方法

| Feature                          | BeanFactory | ApplicationContext |
| -------------------------------- | ----------- | ------------------ |
| bean实例化/注入                  | 支持        | 支持               |
| 实例生命周期管理                 | 不支持      | 支持               |
| 自动BeanPostProcessor注册        | 不支持      | 支持               |
| 自动BeanFactoryPostProcessor注册 | 不支持      | 支持               |
| 方便MessageSource接收            | 不支持      | 支持               |
| 内置的ApplicationEvent发布机制   | 不支持      | 支持               |

MessageSourceAware: 获取Message Source相关文本信息(spirng国际化)

> 重写setMessageSource(MessageSource messageSource)方法

ApplicationEventPublisherAware: 发布事件

> 重写setApplicationEventPublisher(ApplicationEventPublisher applicationEventPublisher)方法
>
> 参考[网页](https://blog.csdn.net/qq_28060549/article/details/81073001)

ResourceLoaderAware: 获取资源加载器, 这样获取外部资源文件

> 重写setResourceLoader(ResourceLoader loader)方法
>
> 参考[网页](https://blog.csdn.net/a617332635/article/details/72235883)

BeanPostProcessor: 是Spring IOC容器提供的一个扩展接口, 在bean初始化方法被调用的前后进行自定义操作

> 需要重写 
>
> postProcessorBeforeInitialization(Object bean, String beanName)
>
> postProcessorAfterInitialization(Object bean, String beanName)

BeanFactoryPostProcessor: 可以管理bean工厂内所有beanDefinition(未实例化)数据, 可以修改属性

> 需要重写 postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory)方法

> BeanFactoryPostProcessor, 与 BeanPostProcessor的区别是: 可以BeanFactoryPostProcessor获取的是bean示例或定义,  同时可以修改bean的属性.

InitializingBean: 提供了bean的初始化方法

> 重写afterPropertiesSet() 方法

> 配置文件中配置init-method同样可以实现初始化方法

> 两种初始化方法的区别:
>
> * 实现InitializingBean接口是直接调用afterPropertiesSet方法, 比通过反射调用init-method指定的方法效率要高, 而init-method方法消除了对spring的依赖
> * 如果afterPropertiesSet方法出错, 则调用init-method指定的方法.

DisposableBean: 提供了bean销毁的方法

> 重写destroy方法
>
> 类似的还有配置文件中配置destroy-method方法