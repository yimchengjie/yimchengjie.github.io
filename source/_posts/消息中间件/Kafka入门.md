---
title: Kafka入门
categories:
  - 消息中间件
tags:
  - 消息中间件
  - Kafka
toc: true
date: 2020-05-27 17:26:17
---
## Kafka入门

Kafka是一款分布式、分区的、多副本的、多订阅者的，基于Zookeeper协调的分布式日志系统（MQ系统），主要应用场景为日志收集系统和消息系统。

## Kafka消息系统

一个消息系统, 负责将数据从一个应用发送到另一个应用, 应用只关心数据, 无需关注数据是如何传递的. 分布式消息传递基于可靠的消息队列. 通过消息队列实现异步,削峰, 解耦.

消息队列一般分为两种模式, 点对点传输和发布订阅模式. Kafka是一种发布订阅模式的消息队列.

### 发布-订阅模式

在发布-订阅模式中, 服务方将消息发布到一个Topic,可以看成是频道. 消费者只要订阅了该频道, 就可以消费其中的数据, 频道中的数据被消费后不会马上删除.

### Kafka架构

#### 服务器: broker

borker为一个Kafka节点, broker上存储了Topic的数据

#### Topic

Topic, 主题 频道, 即消息所属的类别. 该类别称作Topic, 物理上Topic可能分布于多台服务器, 但是逻辑上是同一个, 用户只需要指定Topic即可

#### Partition

一个Topic会被分割为多个Partition, Partition的数量可以为1

#### Producer

生产者, 生产者将消息发布到Topic, broker接收到消息后,存到Partition中

#### Consumer

消费者, 消费者可以从broker中读取数据, 只需要订阅topic

#### Leader

每个Partition可以有多个副本, 但是只有一个Leader, 该Leader负责数据读写

#### Follower

Follower即其他的Partition, 它们跟随Leader,当Leader数据变更时, Follower的数据就位同步, 如果Leader失效, 则会在Follower中重新选取Leader
