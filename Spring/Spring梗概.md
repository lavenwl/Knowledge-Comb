### Spring的设计初衷

* 基于POJO的轻量级和最小侵入性编程
* 通过依赖注入和面向接口松耦合
* 基于切面和惯性进行声明式编程
* 通过切面模板减少样版式代码

### Spring注解编程的演变







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