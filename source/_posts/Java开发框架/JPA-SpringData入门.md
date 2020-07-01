---
title: JPA-SpringData入门
categories:
  - Java开发框架
tags:
  - Spring
  - Java开发框架
  - JPA
toc: true
date: 2019-11-12 21:30:45
---
## JPA-SpringData入门

SpringData JPA是基于Spring的ORM框架,采用JPA规范,底层采用了Hibernate,而Hibernate也是基于JPA规范开发的
它提供了增删改查等在内的功能,且易于扩展

### 组成

1. Repository: 标识接口,表明任何继承它的类都是仓库接口类,方便Spring自动扫描
2. CrudRepository: 继承了Repository, 实现了CRUD的方法
3. PagingAndSortingRepository: 继承了CrudRepository,实现了一组分页排序相关的方法
4. JpaRepository: 继承PagingAndSortingRepository,实现了一组JPA规范的方法
5. JpaSpecificationExecutor: 不属于Repository体系, 它实现了一组JPA规范的查询相关的方法

根据功能,可以看到Repository系列组成的模块, 相当于以前的Mapper层(或者Dao层)
且提供了基础的增删改查,无需手写, 相比Mybatis方便很多

#### JpaRepository

JpaRepository继承了PagingAndSortingRepository,间接也继承了CrudRepository
所以JpaRepository的功能是非常全面的,一般数据访问层的接口实现这个接口
它的定义如下

```java
public interface JpaRepository<T, ID extends Serializable> extends PagingAndSortingRepository<T, ID> {
    List<T> findAll();//查询所有对象，不排序
    List<T> findAll(Sort sort);//查询所有对象，并排序
    <S extends T> List<S> save(Iterable<S> entities);//批量保存
    void flush();//强制缓存与数据库同步
    T saveAndFlush(T entity);//保存并强制同步
    void deleteInBatch(Iterable<T> entities);//批量删除
    void deleteAllInBatch();//删除所有
}
```

### 如何执行SQL

JPA规范下有HQL,JPQL语句, 一般在复杂查询时使用, 但是如果能直接写SQL还是更好的(更熟悉,更可读)

JPA下如何执行SQL语句呢

1. 利用if语句合理拼接SQL
2. createNativeQuery()方法可以创建要执行自然SQL语句查询`Query query=entityManager.createNativeQuery(sql.toString)`
3. query.getSingleResult()方法执行
