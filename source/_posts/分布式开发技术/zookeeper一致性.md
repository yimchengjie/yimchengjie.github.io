---
title: zookeeper一致性
categories:
  - 分布式开发技术
tags:
  - Zookeeper
  - 分布式开发技术
toc: true
date: 2021-05-01 20:57:49
---
## zookeeper一致性

一个zookeeper集群需要保证数据一致性, 他的过程是又leader接受数据修改命令, 将命令发送给其他从服务器,然后自身执行命令.

### 领导者选举算法

首先zookeeper需要有个leader,来保证数据一致性.

leader推荐: 哪一个zookeeper值得推荐
1. 数据越新能力越强,通过日志中的自增id来判断谁的数据更新
2. myid,由启动配置文件指定, 用户可以工具每个机器的性能个性化指定myid来决定leader,

### 二阶段提交(2PC)

