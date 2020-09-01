### 多注册中心

1. 修改配置文件

    ```properties
    dubbo.registries.shanghai.address=zookeeper://192.168.216.128:2181
    dubbo.registries.hunan.address=nacos://192.168.216.128:8848
    ```

2. 修改服务类, 可以选择注册到指定的注册中心上

    ```java
    @DubboService(registry = {"shanghai","hunan"}, version = "1.0")
    ```

3. 修改消费类, 可以选择获取指定注册中心上面的数据

    ```java
    @DubboReference(registry = {"shanghai","hunan"},version = "1.0")
    ```

### 多版本灰度发布

1. 服务类指定发布的版本号

    ```java
    @DubboService(version = "1.0")
    ```

2. 消费类指定请求的版本号

    ```java
    @DubboReference(version = "1.0")
    ```

### 多协议支持

1. 修改服务类注解参数, 支持发布多种协议类型

    ```java
    @DubboService(protocol = {"dubbo","rest"})
    ```

    ```properties
    # Netty ->
    dubbo.protocols.dubbo.name=dubbo
    dubbo.protocols.dubbo.port=-1
    
    
    # jetty (rest需要定义服务类型)
    dubbo.protocols.rest.name=rest
    dubbo.protocols.rest.port=-1
    dubbo.protocols.rest.server=jetty
    ```

    > 如果定义了rest类型的服务, 是否需要添加对应web容器的依赖, 以及rest支持
    >
    > * 添加依赖
    >
    >     ```xml
    >     <dependency> 
    >       <groupId>org.jboss.resteasy</groupId> 
    >       <artifactId>resteasy-jaxrs</artifactId> <version>3.13.0.Final</version> 
    >     </dependency> 
    >     <dependency> 
    >       <groupId>org.jboss.resteasy</groupId> 
    >       <artifactId>resteasy-client</artifactId> <version>3.13.0.Final</version> 
    >     </dependency> 
    >     <dependency> 
    >       <groupId>org.eclipse.jetty</groupId> 
    >       <artifactId>jetty-server</artifactId> <version>9.4.19.v20190610</version> 
    >     </dependency> 
    >     <dependency> 
    >       <groupId>org.eclipse.jetty</groupId> 
    >       <artifactId>jetty-servlet</artifactId> <version>9.4.19.v20190610</version> 
    >     </dependency>
    >     ```
    >
    > * 修改API接口类
    >
    >     ```java
    >     package com.gupaoedu.springboot.dubbo;
    >     
    >     import javax.ws.rs.GET;
    >     import javax.ws.rs.Path;
    >     @Path("/")
    >     public interface ISayHelloService {
    >     
    >         @GET
    >         @Path("/say")
    >         String sayHello(String msg);
    >     }
    >     ```
    >
    >     

2. 修改消费类注解参数, 定义客户端的协议类型

     ```java
    @DubboReference(protocol = "dubbo")
     ```

> dubbo支持的发布协议
>
> * dubbo 默认协议
> * hessian
> * thrift
> * grpc http2.0/ protobuff
> * rest
> * rmi
> * ....

### dubbo负载均衡机制

1. Random(默认)

    可以设置不同的权重, 计算方式是采用的区间权重法

2. roundrobin(轮询)

    所谓轮询就是讲请求依次发布到每一台服务器上, 针对不同服务器的性能差异, 可以对服务器设置不同权重, 来平衡个服务器之间的压力

3. 一致性Hash负载

    默认根据第一个参数进行hash取模, 定位到具体摸一个服务器

4. 最小活跃度

    根据请求处理的吞吐量, 发起一次请求, 计数器+1, 来设置不同的权重, 处理性能越高, 权重也就越高, 反之相同;

5. shortestreponse 负载

    最短响应时间: 计算这些调用程序的权重和数量, 根据响应时间的长短来分配目标服务器的路由权重.

### 集群容错

如果客户端发起一个请求, 报错了, 接下来怎么处理的配置

1. failover cluster(默认): 失败自动重试, 重试其他的服务器,自动切换

    ```java
    @DubboService(cluster="failover", retries=2)
    ```

    > 如果重试不同的服务器, 需要着重处理`幂等`的特性, 避免同一个请求的重复处理

2. failfast cluster: 快速失败

3. failsafe cluster: 安全失败, 如果出现异常, 直接吞掉, 不作处理 如日志处理

4. failback cluster: 失败自动回复, 记录失败请求, 定时重发

5. forKing cluster: 并行调用多个节点, 只要有一个成功返回, 就直接返回结果

6. broadcast cluster: 广播调用, 一个请求调用所有的服务器, 只要其中一个节点报错, 那么就认为这个请求失败.

### Dubbo泛化

发布一个服务, 定义一个服务的接口,对应的消费者也依赖这个接口进行开发, 直接使用. 不需要关心这个接口是怎么触发调用的, 泛化是指生产者与消费者之间没有这样一个公共的接口.

```java
@DubboReference(interfaceName = "com.gupaoedu.springboot.dubbo.springbootdubbosampleprovider.services.IDemoService",generic = true,check = false)
    GenericService genericService;

    @GetMapping("/demo")
    public String demo(){
        Map<String,Object> user=new HashMap<>();
        user.put("",""); //key表达user对象中的属性，value表达属性的值
        return genericService.$invoke("getTxt",new String[0],null).toString();
    }
```

### 服务降级

dubbo提供mock服务, 如果调用失败, 统一返回一个默认的返回

```java
@DubboReference(registry = {"shanghai","hunan"},
            mock = "com.gupaoedu.springboot.dubbo.springbootdubbosampleconsumer.MockSayHelloService",
            timeout = 500,
            cluster = "failfast",check = false,retries = 5)
    ISayHelloService sayHelloService;

    @GetMapping("/say")
    public String say(){
        return sayHelloService.sayHello("Mic");
    }


public class MockSayHelloService implements ISayHelloService{

    @Override
    public String sayHello(String msg) {
        return "触发服务降级";
    }
}
```

### dubbo主机绑定问题

* 查找环境变量中是否存在启动参数`DUBBO_IP_TO_BIND`=服务注册ip
* 读取配置文件, dubbo.protocols.dubbo.host=服务注册ip
* InetAddress.getLocalHost().getHostAddress()获取本地IP地址
* 通过Socket去链接注册中心, 从而获取本机ip
* 轮询本机网卡, 知道找到合适的ip地址

> 上面获取的ip地址是bindip; 如果需要作为服务注册中心的ip, `DUBBO_OP_TO_REGISTRY` -dDUBBO_IP_TO_REGISTRY=ip

### 配置的优先级问题

* 方法层面的配置优先于接口层面的配置, 接口层面的配置优先于全局配置
* 如果级别一样, 客户端的配置优先, 服务端次之

### 多序列化支持

* gRPC protobuf/ json/xml
* dubbo hessian
* webservice /xml
* fst /kryo/ protostuff/gson/avro...

以kryo为例: 

* 添加依赖

 ```xml
 <dependency>
   <groupId>com.esotericsoftware</groupId>
   <artifactId>kryo</artifactId>
   <version>4.0.2</version>
</dependency>
<dependency>
  <groupId>de.javakaffee</groupId>
  <artifactId>kryo-serializers</artifactId>
  <version>0.45</version>
</dependency>
 ```

* 修改配置文件

    ```properties
    dubbo.protocol.serialization=kryo
    ```

### QOS

dubbo 命令行实时运维插件



### 关于DUBBO 调优.......