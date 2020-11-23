---
title: 由浅入深RocketMQ
categories:
  - 消息中间件
tags:
  - 消息中间件
toc: true
date: 2020-11-10 16:36:45
---
## 由浅入深RocketMQ

RocketMQ作为阿里开源的一款高性能、高吞吐量的消息中间件，承载了阿里双十一大部分业务。同时使用Java开发语言，成为了互联网行业首选的消息中间件

### 项目结构
![RocketMQ项目结构](/RocketMQ项目结构.png)
+ broker: 接收生产者发来的消息，消费者从这里消费消息
+ client: 提供给他人使用的发送、接受消息的客户端API
+ namesrv: NameServer实现相关类，类似于Zookeeper，这里保存着消息的TopicName，队列等运行时的元信息
+ filter: 消息过滤相关基础类
+ filtersrv: 消息过滤服务器实现相关类（filter启动进程）
+ remoting: 远程通信模块，基于netty、fastjson序列化
+ store: 消息、索引存储实现
+ logappender: 日志实现相关类
+ common: 通用类、方法、数据结构
+ dev: 开发者信息（非源码）
+ distribution: 部署实例文件夹（非源码）
+ example: RocketMQ示例代码
+ openmessaging: 消息开放标准
+ srvutil: 服务器工具类
+ style: checkstyle相关类
+ test: 测试类
+ tools: 工具、监控命令

主要有四大核心模块: **NameServer、Broker、Producer、Consumer**

### 路由中心NameServer

NameServer类似于Zookeeper，指引消费者找到提供者，完成网络通信

消息中间件的设计思路是基于主题的发布订阅机制，生产者发送某一主题的消息到消息服务器，消息服务器负责把消息持久化，消费者订阅感兴趣的主题，消息服务器根据订阅信息，将消息推送到消费者或者消费者主动拉去消息。为了避免消息服务器的单点故障导致系统瘫痪，一般会部署多个消息服务器共同承担消息存储。那生产者如何知道消息要发送到哪一节点？如果某一台消息服务器宕机，那么生产者如何在不重启的情况下进行感知。NameServer就此而生。

#### NameServer启动流程

