---
title: 深入理解MyBatis（三）
categories:
  - Java开发框架
tags:
  - Java开发框架
  - Mybatis
date: 2020-09-24 20:01:38
toc: true
---
## 深入理解MyBatis（三）

### 插件模块

插件是一种常见的扩展方式，Mybatis也提供了插件的功能。虽然叫插件，但实际上就是拦截器（Interceptor）

#### Interceptor

Mybatis允许用户使用自定义拦截器对SQL语句执行过程中的某一点进行拦截。默认情况下，Mybatis允许拦截Executor、ParameterHandler、ResultSetHandler以及StatementHandler方法，具体方法如下
 + Executor中的update()、query()、flushStatements()、commit()、rollback()、getTransaction()、close()、isClosed()
 + ParammeterHandler中的getParameterObject()、setParameters()
 + ResultSetHandler中的handleResultSets()、handleOutputParameters()
 + StatementHandler中的prepare()、parameterize()、batch()、update()

Mybatis中使用拦截器需要继承Interceptor接口，它的定义如下

    ```java
    public interface Interceptor {

    Object intercept(Invocation invocation) throws Throwable;

    default Object plugin(Object target) {
        return Plugin.wrap(target, this);
    }

    default void setProperties(Properties properties) {
        // NOP
    }

    }
    ```

除了继承Interceptor外，自定义的拦截器还需要使用@Intercepts和@Signature两个注解。
@Intercepts中指定@Signature注解列表。每个@Signature注解中都标识了该插件需要拦截的方法信息
具体如下

    ```java
    @Intercepts({
            // type指定类名，method指定方法名，args指定参数。 一个@Signature注解通过三个属性确定唯一方法
            @Signature(type = Executor.class, method = "query", args = {
                    MappedStatement.class,
                    Object.class,
                    RowBounds.class,
                    ResultHandler.class
            }),
            @Signature(type = Executor.class,method = "update", args = {
                    MappedStatement.class,
                    Object.class
            })
    })
    public class ExamplePlugin implements Interceptor {
        @Override
        public Object intercept(Invocation invocation) throws Throwable {
            return null;
        }
    }
    ```

定义完成以后，需要在配置中添加对该拦截器的配置

    ```xml
    <plugins>
        <plugin interceptor="mybatis.ExamplePlugin">
    </plugins>
    ```

至此，一个自定义的拦截器就配置好了。在mybatis初始化过程中，会对plugin节点解析，最后将Interceptor添加到Configuration中的interceptorChain中，里面使用List保存了所有拦截器.采用责任链模式处理

    ```java
    public class InterceptorChain {
        private final List<Interceptor> interceptors = new ArrayList<>();
        public Object pluginAll(Object target) {
            for (Interceptor interceptor : interceptors) {
             target = interceptor.plugin(target);
            }
            return target;
        }
        public void addInterceptor(Interceptor interceptor) {
            interceptors.add(interceptor);
        }
        public List<Interceptor> getInterceptors() {
            return Collections.unmodifiableList(interceptors);
        }
    }
    ```

#### 应用场景

##### 分页插件

用户可以添加自定义的拦截器，拦截Executor.query方法，从参数RowBounds中获取记录的开始位置，然后获取BoundSql参数，修改待执行的SQL语句，拼接"limit offset,length"，来实现分页功能

以PageHelper为例

```java
@SuppressWarnings({"rawtypes", "unchecked"})
@Intercepts(
        {
                @Signature(type = Executor.class, method = "query", args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class}),
                @Signature(type = Executor.class, method = "query", args = {MappedStatement.class, Object.class, RowBounds.class, ResultHandler.class, CacheKey.class, BoundSql.class}),
        }
)
public class PageInterceptor implements Interceptor {
    // Dialect对象，针对不同的数据库，采用不同的实现策略
    private volatile Dialect dialect;
    private String countSuffix = "_COUNT";
    protected Cache<String, MappedStatement> msCountMap = null;
    private String default_dialect_class = "com.github.pagehelper.PageHelper";
    
    // 具体的拦截方法，通过jdk动态代理创建代理对象
    @Override
    public Object intercept(Invocation invocation) throws Throwable {
        try {
            // 从拦截的方法中获取被拦截的方法参数列表，然后进行类型转换
            Object[] args = invocation.getArgs();
            MappedStatement ms = (MappedStatement) args[0];
            Object parameter = args[1];
            RowBounds rowBounds = (RowBounds) args[2];
            ResultHandler resultHandler = (ResultHandler) args[3];
            // 获取代理目标
            Executor executor = (Executor) invocation.getTarget();
            CacheKey cacheKey;
            BoundSql boundSql;
            //由于逻辑关系，只会进入一次
            if (args.length == 4) {
                //4 个参数时
                boundSql = ms.getBoundSql(parameter);
                cacheKey = executor.createCacheKey(ms, parameter, rowBounds, boundSql);
            } else {
                //6 个参数时
                cacheKey = (CacheKey) args[4];
                boundSql = (BoundSql) args[5];
            }
            checkDialectExists();

            List resultList;
            //调用方法判断是否需要进行分页，如果不需要，直接返回结果
            if (!dialect.skip(ms, parameter, rowBounds)) {
                //判断是否需要进行 count 查询
                if (dialect.beforeCount(ms, parameter, rowBounds)) {
                    //查询总数
                    Long count = count(executor, ms, parameter, rowBounds, resultHandler, boundSql);
                    //处理查询总数，返回 true 时继续分页查询，false 时直接返回
                    if (!dialect.afterCount(count, parameter, rowBounds)) {
                        //当查询总数为 0 时，直接返回空的结果
                        return dialect.afterPage(new ArrayList(), parameter, rowBounds);
                    }
                }
                // 执行分页查询
                resultList = ExecutorUtil.pageQuery(dialect, executor,
                        ms, parameter, rowBounds, resultHandler, boundSql, cacheKey);
            } else {
                //rowBounds用参数值，不使用分页插件处理时，仍然支持默认的内存分页
                resultList = executor.query(ms, parameter, rowBounds, resultHandler, cacheKey, boundSql);
            }
            return dialect.afterPage(resultList, parameter, rowBounds);
        } finally {
            dialect.afterAll();
        }
    }
}
```


