---
title: Redis入门
categories:
  - 缓存中间件
tags:
  - 缓存中间件
  - Redis
toc: true
date: 2019-05-12 14:32:23
---

## Redis 入门

Redis 是一套缓存中间件,是基于内存的数据库,采用 key-value 的形式存储数据,底层由 C 语言开发

以前的数据库读写,需要到硬盘中读取数据,速度相比内存是慢很多的.
Redis 直接存储在内存中,读取速度快,而且 Redis 实现了分布式缓存

### Redis 安装与配置

下载安装

```shell
$sudo apt-get update
$sudo apt-get install redis-server
```

启动服务(默认配置)

```shell
redis-server
```

打开 Redis

```shell
redis-cli
```

redis.conf 是一个默认的配置文件。我们可以根据需要使用自己的配置文件。

### Redis 数据类型

Redis 支持物种数据类型:string(字符串),hash(哈希表),list(列表),set(集合)及 zset(sorted set 有序集合)

#### String(字符串)

redis的字符串和其他编程语言的键值存储提供的字符串非常相似
![string](/string.png)

string 是 redis 最基本的类型,string 类型是二进制安全的,这意味着 redis 的 string 可以包含任何数据. 最大能存储 512MB

#### linkedlist(列表/队列)

redis的列表可执行的操作和很多编程语言类似，`LPUSH`命令和`RPUSH`命令分别是将元素推入列表左端和右端。`LPOP`和`RPOP`分别用于从列表的左端和右端弹出元素。`LINDEX`命令用于获取列表在给定位置的元素，`LRANGE`用于获取给定范围的所有元素
![linkedList](/linkedList.png)

#### Set(集合)

redis 中的 set 是 String 类型的无序集合,且没有重复值,使用`SADD`将元素添加到集合，`SREM`从集合中移除元素，`SISMEMBER`检查元素是否存在集合中，`SMEMBERS`获取集合所有的元素
![set](/set.png)

#### zset(有序结合)

zeset 也是 String 类型的集合,也没有重复值 但它有序。它的每一个成员都需要指定一个浮点数的分数，用于排序。使用`ZADD`将元素添加到集合，`ZREM`从集合中移除元素，`ZRANGE`用于获取给定范围的所有元素，`ZRANGEBYSCORE`用于获取给定分数内的所有元素
![zset](/zset.png)

#### hash(哈希表)

redis 的 hash 是一个键值对集合, 在 redis 中的关系是 key 对应 value,value 中又是键值对. 哈希表就像是一个微缩版的redis。
`HSET`在hash中关联给定的键值对，`HGET`获取给定哈希key的值，`HGETALL`获取所有键值对，`HDEL`删除给定key
![hash](/hash.png)

### Java 使用 Redis

Redis 在 Web 开发中往往用作缓存,存储需要高速读写的数据

Java 使用 Redis 需要用到驱动包 jedis.jar

连接 redis 服务器

```java
public static void main(String[] args){
    Jedis jedis = new Jedis("000.000.000.000", 6379, 100000);
    jedis.set("test","test");
    jedis.get("test");
    jedis.close();
}
```
