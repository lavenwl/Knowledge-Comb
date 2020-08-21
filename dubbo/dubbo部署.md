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

        

    5. 编写基础代码

2. 创建一个普通maven工程: dubbo-spring-server

    1. 引入dubbo依赖
    2. 配置spring中的dubbo相关配置







### 集成SpringCloudAlibaba



### 单独集成Springboot, 与dubbo框架进行使用

