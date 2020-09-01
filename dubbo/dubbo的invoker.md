invoker翻译成中文是调用器, 它在dubbo中是一个比较重要的领域对象, 最核心的是服务的发布与调用中, 都是以invoker的形态存在

invoker本质上是一个代理, 经过层层包装最终进行发布,最终发布出去的invoker也不单纯是一个代理, 而是多层包装后的调用器

invoker来自于ProxyFactory类, 而此类为扩展点, 所以我可以找到他的默认实现是javassist.

> javassist是一个动态类库, 用来创建动态代理