### 单独部署dubbo继承Spring框架

1. 新建一个普通的maven工程: `spring-dubbo-server`

    1. 创建一个普通的maven模块: `spring-dubbo-server-api`

        1. 创建一个借口:`com.laven.spring.dubbo/ILoginService.java`

            ```java
            package com.laven.spring.dubbo;
            
            public interface ILoginService {
                String login(String username, String password);
            }
            ```

        2. maven:install进行打包

    2. 创建一个普通maven模块: `spring-dubbo-server-provider`

    3. 引入dubbo依赖: 

        ```xml
        <dependency>
          <groupId>com.laven.spring.dubbo</groupId>
          <artifactId>spring-dubbo-api</artifactId>
          <version>1.0-SNAPSHOT</version>
        </dependency>
        
        <dependency>
          <groupId>org.apache.dubbo</groupId>
          <artifactId>dubbo</artifactId>
          <version>2.7.8</version>
        </dependency>
        ```

    4. 创建配置文件: `resources/META-INF/spring/application.xml`

        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans"
               xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
               xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"			xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans.xsd        http://code.alibabatech.com/schema/dubbo        http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
        
            <dubbo:application name="dubbo-server" />
          	<!-- 注册中心必须配置, 即使不适用, 也要这样配置 -->
            <dubbo:registry address="N/A" />
          	<!-- dubbo服务的发布的协议及端口 -->
            <dubbo:protocol name="dubbo" port="20880" />
        		<!-- 发布服务的名称以及具体实现 -->
            <dubbo:service interface="com.laven.spring.dubbo.ILoginService" ref="loginService" />
        <!-- spring中的一个普通对象, 用于实现发布的服务 --> 
            <bean id="loginService" class="com.laven.spring.dubbo.LoginServiceImpl" />
          
        </beans>
        ```

    5. 编写ILogin接口的实现类ILoginServiceImpl

        ```java
        
        public class LoginServiceImpl implements ILoginService{
            @Override
            public String login(String username, String password) {
                if (username.equals("admin") && password.equals("admin")) {
                    return "SUCCESS";
                }
                return "FAILED";
            }
        }
        ```

    6. 启动服务

2. 创建一个普通的maven工程: spring-dubbo-client

    1. 添加依赖

        ```xml
         <dependency>
              <groupId>org.apache.dubbo</groupId>
              <artifactId>dubbo</artifactId>
              <version>2.7.8</version>
            </dependency>
        
            <dependency>
              <groupId>com.laven.spring.dubbo</groupId>
              <artifactId>spring-dubbo-api</artifactId>
              <version>1.0-SNAPSHOT</version>
            </dependency>
        ```

    2. 添加配置文件META-INF/spring/application.xml

        ```xml
        <?xml version="1.0" encoding="UTF-8"?>
        <beans xmlns="http://www.springframework.org/schema/beans"
               xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
               xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
               xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans.xsd        http://code.alibabatech.com/schema/dubbo        http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
        
            <dubbo:application name="dubbo-client" />
        
            <dubbo:registry address="N/A" />
            <dubbo:reference id="loginService"             interface="com.laven.spring.dubbo.ILoginService"                                    url="dubbo://192.168.99.113:20880/com.laven.spring.dubbo.ILoginService"
        />
            
        
        </beans>
        ```

        

    3. 写测试类

        ```java
        public class App 
        {
            public static void main( String[] args )
            {
                ILoginService loginService = null;
                ApplicationContext context = new ClassPathXmlApplicationContext("classpath:META-INF/spring/application.xml");
                loginService = context.getBean(ILoginService.class);
        
                System.out.println( loginService.login("admin", "admin") );
            }
        }
        ```

    4. 启动测试是否链接到服务端

> 如果需要使用注册中心
>
> 添加响应的依赖如:
>
> ```xml
> <dependency>
>   <groupId>com.alibaba.nacos</groupId>
>   <artifactId>nacos-client</artifactId>
>   <version>1.2.1</version>
> </dependency>
> ```
>
> 对应的配置修改为:
>
> ```xml
> <!--    <dubbo:registry address="N/A" />-->
>     <dubbo:registry address="nacos://localhost:8848" />
> 
> <!--    <dubbo:reference id="loginService"-->
> <!--                     interface="com.laven.spring.dubbo.ILoginService"-->
> <!--                     url="dubbo://192.168.99.113:20880/com.laven.spring.dubbo.ILoginService"-->
> <!--    />-->
>     <dubbo:reference id="loginService"
>                      interface="com.laven.spring.dubbo.ILoginService"
>     />
> ```

### 集成SpringCloudAlibaba

1. 创建一个普通maven项目: dubboExample

2. 新建一个API模块: dubbo-api

    * 新建一个接口:

        ```Java
        public interface ISayHelloService {
            String sayHello(String msg);
        }
        ```

    * 打包jar

3. 新建一个服务提供者springboot模块: provider

    * 添加依赖

        ```xml
        <dependency>
                    <groupId>com.laven</groupId>
                    <version>1.0-SNAPSHOT</version>
                    <artifactId>dubbo-api</artifactId>
                </dependency>
                <dependency>
                    <groupId>com.alibaba.cloud</groupId>
                    <artifactId>spring-cloud-starter-dubbo</artifactId>
                    <version>2.2.1.RELEASE</version>
                </dependency>
                <dependency>
                    <groupId>com.alibaba.cloud</groupId>
                    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
                    <version>2.2.1.RELEASE</version>
                </dependency>
        ```

    * 新建一个sayHello的实现

        ```java
        package com.laven.provider;
        
        import com.laven.ISayHelloService;
        import org.apache.dubbo.config.annotation.Service;
        
        @Service
        public class SayHelloServiceImpl implements ISayHelloService {
            @Override
            public String sayHello(String msg) {
                return "From Provider Message!!!";
            }
        }
        ```

    * 修改application.xml

        ```xml
        spring.application.name=provider
        dubbo.scan.base-packages=com.laven.provider
        
        dubbo.protocol.name=dubbo
        dubbo.protocol.port=20880
        
        springcloud.nacos.discovery.server-addr=localhost:8848
        ```

    * 启动服务者模块

4. 新建一个消费者Springboot模块: consumer

    * 引入依赖

        ```xml
        <dependency>
                    <groupId>com.alibaba.cloud</groupId>
                    <artifactId>spring-cloud-starter-dubbo</artifactId>
                    <version>2.2.1.RELEASE</version>
                </dependency>
                <dependency>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-web</artifactId>
                </dependency>
                <dependency>
                    <groupId>com.laven</groupId>
                    <version>1.0-SNAPSHOT</version>
                    <artifactId>dubbo-api</artifactId>
                </dependency>
                <dependency>
                    <groupId>com.alibaba.cloud</groupId>
                    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
                    <version>2.2.1.RELEASE</version>
                </dependency>
        ```

    * 编写测试类

        ```java
        package com.laven.consumer;
        
        import com.laven.ISayHelloService;
        import org.apache.dubbo.config.annotation.Reference;
        import org.springframework.boot.SpringApplication;
        import org.springframework.boot.autoconfigure.SpringBootApplication;
        import org.springframework.web.bind.annotation.GetMapping;
        import org.springframework.web.bind.annotation.RestController;
        
        @RestController
        @SpringBootApplication
        public class ConsumerApplication {
        
            public static void main(String[] args) {
                SpringApplication.run(ConsumerApplication.class, args);
            }
        
            @Reference
            ISayHelloService sayHelloService;
        
            @GetMapping("/say")
            public String say() {
                return sayHelloService.sayHello("mic");
            }
        }
        ```

    * 修改application.xml

        ```xml
        spring.application.name=consumer
        server.port=8081
        springcloud.nacos.discovery.server-addr=localhost:8848
        ```

    * 启动消费者模块

    

### 单独集成Springboot, 与dubbo框架进行使用

1. 新建一个普通的maven项目: springboot-dubbo

2. 新建一个普通的maven模块: api 作为统一服务协议

    * 新建一个接口: com.laven.springboot.dubbo.api.ISayHelloService

        ```java
        package com.laven.springboot.dubbo.api;
        
        public interface ISayHelloService {
            public String say();
        }
        ```

    * 打包

3. 新建一个Springboot的项目: provider 作为生产者

    1. pom.xml文件中添加依赖

        ```xml
        <dependency>
          <groupId>org.apache.dubbo</groupId>
          <artifactId>dubbo-spring-boot-starter</artifactId>
          <version>2.7.7</version>
        </dependency>
        <dependency>
          <groupId>com.alibaba.nacos</groupId>
          <artifactId>nacos-client</artifactId>
          <version>1.2.1</version>
        </dependency>
        
        <dependency>
          <groupId>com.laven.springboot.dubbo</groupId>
          <artifactId>api</artifactId>
          <version>1.0-SNAPSHOT</version>
        </dependency>
        ```

        > 注意: 这里添加的dubbo是dubbo为Springboot写的启动插件`org.apach.dubbo/dubbo-spring-boot-starter`, 这里要与springcloud项目集成里面的`com.alibaba.cloud/spring-cloud-starter-dubbo`分开

    2. 编写ISayHelloService接口的实现`com.laven.springboot.dubbo.provider.SayHelloServiceImpl`

        ```java
        package com.laven.springboot.dubbo.provider;
        
        import com.laven.springboot.dubbo.api.ISayHelloService;
        import org.apache.dubbo.config.annotation.DubboService;
        import org.springframework.beans.factory.annotation.Autowired;
        import org.springframework.beans.factory.annotation.Value;
        
        @DubboService
        public class SayHelloServiceImpl implements ISayHelloService {
        
            @Override
            public String say() {
                return "this is from Provider and name";
            }
        }
        ```

    3. 配置application.xml文件

        ```xml
        spring.application.name=provider
        myName=this is a name
        
        dubbo.registry.address=nacos://localhost:8848
        #dubbo.scan.base-packages=com.laven.springboot.provider
        
        dubbo.protocol.name=dubbo
        dubbo.protocol.port= -1
        ```

    4. 启动provider模块

4. 新建一个Springboot的项目: consumer 作为消费者

    * 添加依赖

        ```xml
        <dependency>
          <groupId>org.apache.dubbo</groupId>
          <artifactId>dubbo-spring-boot-starter</artifactId>
          <version>2.7.7</version>
        </dependency>
        <dependency>
          <groupId>com.alibaba.nacos</groupId>
          <artifactId>nacos-client</artifactId>
          <version>1.2.1</version>
        </dependency>
        
        <dependency>
          <groupId>com.laven.springboot.dubbo</groupId>
          <artifactId>api</artifactId>
          <version>1.0-SNAPSHOT</version>
        </dependency>
        ```

    * 编写消费者代码

        ```Java
        package com.laven.springboot.dubbo.consumer;
        
        import com.laven.springboot.dubbo.api.ISayHelloService;
        import jdk.nashorn.internal.ir.annotations.Reference;
        import org.apache.dubbo.config.annotation.DubboReference;
        import org.springframework.boot.SpringApplication;
        import org.springframework.boot.autoconfigure.SpringBootApplication;
        import org.springframework.web.bind.annotation.GetMapping;
        import org.springframework.web.bind.annotation.RestController;
        
        @RestController
        @SpringBootApplication
        public class ConsumerApplication {
        
        	public static void main(String[] args) {
        		SpringApplication.run(ConsumerApplication.class, args);
        	}
        
        	@DubboReference
        	ISayHelloService sayHelloService;
        
        	@GetMapping("/say")
        	public String say() {
        		return sayHelloService.say();
        	}
        }
        ```

    * 配置application.xml

        ```xml
        spring.application.name=consumer
        server.port=8080
        
        dubbo.registry.address=nacos://localhost:8848
        ```

    * 启动consumer

