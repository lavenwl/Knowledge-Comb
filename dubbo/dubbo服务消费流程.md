* 首先会在DubboAutoConfiguration中配置一个自动装配机制

* 最终执行ReferenceAnnotationBeanPostProcessor中重写的方法`doGetInjectedBean`也就是实现bean依赖注入的方法

    在这个方法中, 注意做了两个事情

    * 注册一个ReferenceBean到SpringIOC容器中
    * 调用getOrCreateProxy返回一个动态代理对象
    * 