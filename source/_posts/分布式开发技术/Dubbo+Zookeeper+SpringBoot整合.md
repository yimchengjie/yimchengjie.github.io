---
title: Dubbo+Zookeeper+SpringBoot整合
categories:
  - 分布式开发技术
tags:
  - Dubbo
  - 分布式开发技术
  - Spring
toc: true
date: 2019-05-24 00:15:49
---
## Dubbo+Zookeeper+SpringBoot整合

------------------------------------

Dubbo作为一款非常高性能轻便的RPC框架,往往搭配着Zookeeper作为服务注册发现中心来使用
SpringBoot在Java开发中也越来越主流,开发非常便利
如何整合Dubbo+Zookeeper+SpringBoot进行分布式开发呢?

### 整合步骤

#### 1. 创建项目

新建一个maven工程dubbo_demo,然后在工程下, 创建三个模块,provider,consumer,common(通用接口jar)

![项目创建](/项目创建.jpg)

在父pom中导入需要的包

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion><groupId>com.ycj</groupId>
    <artifactId>DubboDemo</artifactId>
    <packaging>pom</packaging>
    <version>1.1-SNAPSHOT</version>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.2.4.RELEASE</version>
    </parent>
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
    </properties>
    <modules>
        <module>provider</module>
        <module>common</module>
        <module>consumer</module>
    </modules>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.47</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.alibaba.boot/dubbo-spring-boot-starter -->
        <dependency>
            <groupId>com.alibaba.boot</groupId>
            <artifactId>dubbo-spring-boot-starter</artifactId>
            <version>0.1.0</version>
        </dependency>
        <dependency>
            <groupId>io.netty</groupId>
            <artifactId>netty-all</artifactId>
            <version>4.1.42.Final</version>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.7</version>
        </dependency>
        <!-- ZooKeeper client -->
        <!-- https://mvnrepository.com/artifact/com.101tec/zkclient -->
        <dependency>
            <groupId>com.101tec</groupId>
            <artifactId>zkclient</artifactId>
            <version>0.10</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

common的pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <groupId>com.ycj</groupId>
        <artifactId>DubboDemo</artifactId>
        <version>1.1-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.ycj.dubboDemo</groupId>
    <artifactId>common</artifactId>
    <packaging>jar</packaging>
</project>
```

provider和consumer只需导入common即可

```xml
<dependency>
    <groupId>com.ycj.dubboDemo</groupId>
    <artifactId>common</artifactId>
    <version>1.1-SNAPSHOT</version>
</dependency>
```

#### 2. 编写接口

![编写接口](/编写接口.png)

```java
public interface DemoService {
    public String getDemoString();
}
```

#### 3. 编写提供者

![编写提供者](/编写提供者.png)

```java
// DemoService的实现类
// 这里的@Service是Dubbo的Service,不是spring的
@Service
public class ProviderService implements DemoService {

    @Override
    public String getDemoString() {
        return "ProviderService";
    }
}
```

```java
// 启动类, 注意加载配置
@SpringBootApplication
@ImportResource(locations = {"classpath:provider.xml"})
@EnableDubbo
public class ProviderApplication {

    public static void main(String[] args) {
        SpringApplication.run(ProviderApplication.class, args);
    }

}
```

dubbo提供者配置,注意资源地址`http://code.alibabatech.com/`,Apache的我试了一下不可用(当前pom配置情况下)

```xml
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://code.alibabatech.com/schema/dubbo
       http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
    <dubbo:application name="demo-provider"/>
    <dubbo:registry address="zookeeper://127.0.0.1:2181"/>
    <dubbo:protocol name="dubbo" port="20890"/>
    <bean id="demoService" class="com.ycj.dubboDemo.provider.service.ProviderService"/>
    <dubbo:service interface="com.ycj.dubboDemo.common.service.DemoService" ref="demoService"/>
</beans>
```

#### 4. 编写消费者

![编写消费者](/编写消费者.png)

```java
// 调用类
@Controller
public class consumer {
    @Autowired
    public DemoService demoService;

    @RequestMapping("/test")
    @ResponseBody
    public String test(){
        return demoService.getDemoString();
    }
}
```

```java
// 启动类,注意加载配置
@SpringBootApplication
@ImportResource(locations = {"classpath:consumer.xml"})
@EnableDubbo
public class ConsumerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConsumerApplication.class, args);
    }
}
```

dubbo消费者配置

```xml
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://code.alibabatech.com/schema/dubbo"
       xmlns="http://www.springframework.org/schema/beans"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd
       http://code.alibabatech.com/schema/dubbo
       http://code.alibabatech.com/schema/dubbo/dubbo.xsd">
    <dubbo:application name="demo-consumer"/>
    <dubbo:registry address="zookeeper://127.0.0.1:2181"/>
    <dubbo:reference id="demoService" check="false" interface="com.ycj.dubboDemo.common.service.DemoService"/>
</beans>
```

#### 5. 启动Zookeeper

![启动Zookeeper](/启动Zookeeper.png)

#### 6. 启动提供者

![启动提供者](/启动提供者.png)

#### 7. 启动消费者

![启动消费者](/启动消费者.png)

#### 8. 访问成功

![访问成功](/访问成功.png)
