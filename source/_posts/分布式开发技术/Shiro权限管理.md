---
title: Shiro权限管理
categories:
  - 分布式开发技术
tags:
  - Shiro
  - 权限管理
toc: true
date: 2020-02-07 01:30:49
---

## Shiro权限管理

Shiro是一款强大灵活的安全框架,可以处理身份**认证**,**授权**,**加密**和**会话管理**

### 权限管理概念

一般有用户参与的系统,都需要权限管理,权限管理**实现了对用户访问系统的控制**

1. 用户认证

    用户认证,即用户访问系统时,会被验证用户身份的合法性

2. 用户授权

    在用户认证通过后,系统对用户访问资源进行控制,用户拥有资源的权限时,才可以进行访问

#### 权限管理组成

1. 主体(账号、密码)
2. 权限(权限名称、资源名称、资源访问地址)
3. 角色(角色名称)
4. 角色和权限的关系(角色id、权限id)
5. 主体和角色的关系(主体id、角色id)

#### 分配权限

用户需要分配相应的权限才可以访问相应资源
通常给用户分配资源权限需要将权限信息持久化保存,比如存到关系型数据库

然后实现访问控制

1. 基于角色的访问控制

    ```java
    if(user.hasRole("部门经理")){
        //
    }
    ```

    基于角色的访问控制,不利于代码的维护,扩展性差,因为判断角色的条件都写死在代码中

2. 基于资源的访问控制

    ```java
    if(user.hasPermission ('用户报表查看（权限标识符）')){
        //系统资源内容
        //用户报表查看
    }
    ```

    资源在系统中不变,建议使用基于资源的访问控制

### Shiro架构

1. Subject: 当前用户
2. SecurityManager: 管理啊所有Subject,SecurityManager是Shiro架构的核心
3. Realms: 用户进行权限信息的验证,可以自己实现,它封装了与数据源连接的细节,在配置Shiro的时候,必须指定至少一个Realm来实现认证/授权

#### Shiro认证过程

1. 创建SecurityManager
2. Subject提交认证
3. SecurityManager认证
4. Authenticator认证
5. Realm认证

```java
public class AuthenticationTest {
    // 框架提供默认Realm
    SimpleAccountRealm simpleAccountRealm = new SimpleAccountRealm();

    @Before
    public void addUser(){
        simpleAccountRealm.addAccount("ycj","123456","admin","user");
    }

    @Test
    public void testAuthentication(){
        // 1. 创建SecurityManager
        DefaultSecurityManager securityManager=new DefaultSecurityManager();
        //   1.1 为SecurityManager设置Realm
        securityManager.setRealm(simpleAccountRealm);

        // 2. 主体提交认证请求
        //   2.1 为SecurityUtils注入securityManager
        SecurityUtils.setSecurityManager(securityManager);
        //   2.2 获取当前Subject
        Subject subject=SecurityUtils.getSubject();
        //   2.3 收集用户名和凭证
        UsernamePasswordToken token = new UsernamePasswordToken("ycj","123456");

        //   2.4 将用户名和凭证提交到SecurityManager,获取认证
        subject.login(token);
        //判断认证状态
        System.out.println("isAuthenticated:" + subject.isAuthenticated());
        //判断是否有以下角色权限
        subject.checkRoles("admin","user");

        //退出,取消授权
        subject.logout();
        System.out.println("isAuthenticated:" + subject.isAuthenticated());
    }
}
```
