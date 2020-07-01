---
title: Shiro入门
categories:
  - Java开发框架
tags:
  - Java开发框架
  - Shiro
  - 安全框架
date: 2020-06-27 12:13:49
toc: true
---

## Shiro入门

--------------

Shiro是Java的一个安全框架,它可以完成**认证,授权,加密,会话管理,与Web集成,缓存等**

### Shiro功能组成

Authentication: 身份认证/登录, 安全验证
Authorization: 授权/权限验证
Session Manager: 会话管理, 一次登录就是一次会话,没有退出前,所有信息都会保存到这次会话中
Cryptograpyh: 加密
Web Support: 可以集成到Web环境
Caching: 支持缓存
Concurrency: 支持多线程的并发验证,支持权限的传播(把权限从一个线程传播到它开启的另一个线程中)
Testing: 测试支持
Run As: 允许一个用户假装成另一个用户的身份进行访问
Remember Me: 记住我,即登录后下次不需要重新登录

### Shiro架构

**Subject**: 应用代码直接交互的对象是Subject, Shiro堆外的核心API就是Subject, Subject代表了当前用户
**SecurityManager**: 安全管理器,所有的安全操作都交给SecurityManager交互, 它管理着所有的Subject, 它是Shrio的核心, 负责与其他组件交互.可以理解为SpringMVC中的DispatcherServlet
**Realm**: Shiro从Realm获取安全数据(用户,角色,权限),SecurityManager从Realm中获取响应的用户进行比较然后确定用户是否合法,可以把Realm理解为DataSource