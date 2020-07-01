---
title: 初识Dubbo
categories:
  - 分布式开发技术
tags:
  - Dubbo
  - 分布式开发技术
toc: true
date: 2019-05-22 16:15:49
---
## 初识Dubbo

-----------------------

随着互联网的发展,网站应用的规模不断扩大,常规的垂直应用架构以及无法应对(应用拆分的越来越细,模块之间的交互在所难免), 分布式服务架构以及流动计算架构势在必行

**分布式服务架构**: 将应用的核心业务抽取出来,作为独立的服务,逐渐形成稳定的服务中心,使前端应用能更快速的响应多变的市场需求. 此时,用于提高业务复用以及整合的分布式服务框架(RPC)就成了关键

**流动计算架构**: 当服务越来越多, 部分小服务就会有资源浪费的问题,此时需要增加一个调度中心基于访问压力实时管理集群容量,提高集群利用率,此时, 用于提高机器利用率的资源调度和治理中心(SOA)是关键

Dubbo是阿里巴巴开源的一款**高性能、轻量级的开源JavaRPC框架**

它体统了三大核心能力：**面向接口的远程方法调用**、**智能容错和负载均衡**、以及**服务自动注册和发现**

### Dubbo架构

![Dubbo架构](/dubbo-architecture.jpg)

节点角色
节点|角色说明
-|-
Provider|暴露服务提供方
Consumer|调用远程服务的服务消费方
Registry|服务注册与发现的注册中心
Monitor|统计服务的调用次数和调用时间的监控中心
Container|服务运行容器

1. Container负责启动,加载,运行Provider
2. Provider在启动时,向Registry注册自己提供的服务
3. Consumer在启动时,向Registry订阅自己所需的服务
4. Registry返回Provider提供的地址列表给Consumer,一旦列表有变更,Registry将基于长连接(TCP)推送变更数据给Consumer
5. Consumer,从Provider提供的地址列表中,基于软负载均衡算法,选取一台服务器进行调用,如果调用失败,再选另一台调用
6. Comsumer和Provider,在内存中累计调用次数和调用时间,定时每分钟发送一次统计数据到Monitor

### Dubbo快速启动

Dubbo采用全Spring的配置方式,透明化接入应用,对应用没有任何API侵入,只需要用Spring加载Dubbo的配置即可

#### 服务提供者

1. 定义服务接口

    ```java
    public interface DemoService {
        String sayHello(String name);
    }
    ```

2. 在服务提供方实现接口

    ```java
    public class DemoServiceImpl implements DemoService {
        public String sayHello(String name) {
            return "Hello " + name;
        }
    }
    ```

3. 在Spring配置(provider.xml)中声明暴露服务

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
        xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans-4.3.xsd        http://dubbo.apache.org/schema/dubbo        http://dubbo.apache.org/schema/dubbo/dubbo.xsd">
        <!-- 提供方应用信息，用于计算依赖关系 -->
        <dubbo:application name="hello-world-app"  />

        <!-- 使用multicast广播注册中心暴露服务地址 -->
        <dubbo:registry address="multicast://224.5.6.7:1234" />

        <!-- 用dubbo协议在20880端口暴露服务 -->
        <dubbo:protocol name="dubbo" port="20880" />

        <!-- 声明需要暴露的服务接口 -->
        <dubbo:service interface="org.apache.dubbo.demo.DemoService" ref="demoService" />

        <!-- 和本地bean一样实现服务 -->
        <bean id="demoService" class="org.apache.dubbo.demo.provider.DemoServiceImpl" />
    </beans>
    ```

4. 加载Spring配置文件

    ```java
    public class Provider {
        // 程序启动入口
        public static void main(String[] args) throws Exception {
            ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(new String[] {"http://10.20.160.198/wiki/display/dubbo/provider.xml"});
            context.start();
            System.in.read(); // 按任意键退出
        }
    }
    ```

#### 服务消费者

1. 通过Spring配置引用远程服务

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
        xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans-4.3.xsd        http://dubbo.apache.org/schema/dubbo        http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

        <!-- 消费方应用名，用于计算依赖关系，不是匹配条件，不要与提供方一样 -->
        <dubbo:application name="consumer-of-helloworld-app"  />

        <!-- 使用multicast广播注册中心暴露发现服务地址 -->
        <dubbo:registry address="multicast://224.5.6.7:1234" />

        <!-- 生成远程服务代理，可以和本地bean一样使用demoService -->
        <dubbo:reference id="demoService" interface="org.apache.dubbo.demo.DemoService" />
    </beans>
    ```

2. 加载Spring配置,并调用远程服务

    ```java
    public class Consumer {
        public static void main(String[] args) throws Exception {
            ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext(new String[] {"http://10.20.160.198/wiki/display/dubbo/consumer.xml"});
            context.start();
            DemoService demoService = (DemoService)context.getBean("demoService"); // 获取远程服务代理
            String hello = demoService.sayHello("world"); // 执行远程方法
            System.out.println( hello ); // 显示调用结果
        }
    }
    ```
