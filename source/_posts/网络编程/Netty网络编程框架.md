---
title: Netty网络编程框架
categories:
  - 网络编程
tags:
  - 网络编程
  - Netty
toc: true
date: 2019-05-07 20:59:55
---

## Netty 网络编程框架

Netty 是一个高性能、高可扩展性的异步事件驱动的网络应用程序框架，它极大的简化了 TCP 和 UDP 客户端和服务器开发等网络编程

Netty 支持 BIO、NIO，支持各种协议

### Reactor 线程模型

Reactor 模型也叫 Dispatcher 模式,服务器接收多路请求,并同步反派给请求对应的线程

### Netty 线程模型

Netty 线程模型基于主从 Reactor 线程模型,有多个 Reactor

### Netty 模块组件

- **Bootstrap 引导**: 一个 Netty 应用通常由一个 Bootstrap 开始,主要作用是配置整个 Netty 程序,串联各个组件,Netty 中 Bootstrap 类是客户端程序的启动引导类,ServerBootstrap 是服务端启动引导类
- **Future ChannelFuture**: 可以注册一个监听，当操作执行成功或失败时监听会自动触发注册的监听事件。
- **Channel**: 网络通信通道,发送接收数据
- **Selector**: 实现 I/O 多路复用,一个 Selector 线程可以监听多个线程的 Channel 事件
- **NioEventLoop**: 类似线程池,维护了一个线程和任务队列,支持异步提交执行,线程启动时,会调用 run()方法,执行 IO 任务和非 IO 任务
- **NioEventLoopGroup**: 用于管理 eventLoop 的生命周期,可以理解为大线程池,内部维护了一组小线程池
- **ChannelHandler**: 处理 IO 事件或者拦截 IO 操作,并转发到 ChannelPipeline(业务处理链)中
  Handler 可以被共享,但是要注意防止共享变量,比如解码器是不能被共享的
  耗时的业务不要放在 Handler,要单独交给指定的线程池中
- **ChannelHandlerContext**: 保存 Channel 相关的上下文信息,每一个 ChannelHandlerContext 关联一个 ChannelHandler
- **ChannelPipline**: 责任链,业务处理链,保存了 ChannelHandler 的 List,用于处理或拦截 Channel 的入站事件或者出站操作.

![netty流程](netty流程.jpg)

[netty 服务端处理用户请求流程图解](https://luan.ma/post/netty-flow/)
