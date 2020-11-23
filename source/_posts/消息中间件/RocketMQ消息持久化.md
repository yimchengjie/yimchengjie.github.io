---
title: RocketMQ消息持久化
categories:
  - 消息中间件
tags:
  - 消息中间件
toc: true
date: 2020-11-18 16:36:45
---
## RocketMQ消息持久化

持久化，一是防止宕机时的数据丢失，二是可以在高峰期先将数据刷盘，然后再慢慢处理。

------------

### MQ消息持久化的一般思路

1. 不持久化，比如redis、ZeroMQ，由于分布式缓存的读写能力优于DB，如果你的需求是快产快消的即时消费场景,并且生产的消息立即被消费者消费掉，并且看中速度，那么用这种方式也ok。
2. 数据库DB，Apache下的ActiveMQ支持数据库的持久化方式，可以实现JDBC消息存储，由于关系型数据库在数据量很大的情况下的IO读写能力会出现瓶颈，而且非常依赖于数据库。
3. 文件系统，也就是RocketMQ采用的持久化方式，且主流的kafka、rabbitMQ也是采用这种方式，它是将消息刷盘到服务器的文件系统中。它能提供高可靠、高性能，除非MQ挂了或者服务器宕机，不然不会出现无法持久化的问题。

### 存储文件

Commitlog文件、ConsumeQueue文件、IndexFile文件
![](/消息中间件/RocketMQ消息持久化/RocketMQ持久化架构.jpg)

#### Commitlog文件

Commitlog是消息存储的物理文件，为了提高写入的效率，采取顺序写的策略（磁盘的顺序写入不需要重新寻址，效率很高，才机械硬盘中会更明显）。为了实现顺序写，RocketMQ把所有的主题的消息都会存储到同个Commitlog文件中。
CommitLog 中的文件默认大小为 1G，可以动态配置；当一个文件写满以后，会生成一个新的 CommitLog 文件。

##### CommitLog条目

![CommitLog条目](/消息中间件/RocketMQ消息持久化/commitLogItem.png)

1. TOTALSIZE: 该消息条目总长度，4字节
2. MAGICCODE: 魔法值，固定0xdaa320a7，4字节
3. BODYCRC: 消息体crc校验码，4字节
4. QUEUEID: ComsumeQueue消息消费队列ID，4字节
5. FLAG: 消息FLAG，预留给消费者的标识位，4字节
6. QUEUEOFFSET: 消息在ComsumeQueue的偏移量，8字节
7. PHYSICALOFFSET: 消息在CommitLog文件中的偏移量，8字节
8. SYSFLAG: 消息系统FLAG，例如是否压缩、是否有事务消息，4字节
9. BORNTIMESTAMP: 消息产生者调用消息发送API的时间戳，8字节
10. BORNHOST: 消息发送者IP、端口号，8字节
11. STORETIMESTAMP: 消息存储时间戳，8字节
12. STOREHOSPTADDRESS: Broker服务器IP+端口号，8字节
13. RECONSUMETIMES: 消息重试次数，4字节
14. Prepare Transaction Offset: 事务消息物理偏移量，8字节
15. BodyLength: 消息体长度，4字节
16. Body: 消息体内容
17. TopicLength: 主题存储长度，主题名称不能超过255个字符，1字节
18. Topic: 主题内容
19. PropertiesLength: 消息属性长度，表示消息属性长度不能超过65536个字符，2字节
20. Properties: 消息属性

#### ConsumeQueue消息逻辑队列

消息逻辑队列文件，每个消息主题会有多个消息消费队列（MessageQueue），每个消息队列会有一个消息文件。它保存了该MessageQueue的所有消息在CommitLog文件中的物理位置（offset偏移量）

#### IndexFile

IndexFile（索引文件）提供了一种可以通过key或时间区间来查询消息的方法。固定的单个IndexFile文件大小约为400M，一个IndexFile可以保存 2000W个索引，IndexFile的底层存储设计为在文件系统中实现HashMap结构，故rocketmq的索引文件其底层实现为hash索引。

### 持久化逻辑（存储流程）

1. 当Broker停止工作或者是从服务器不支持写时拒接消息写入，或者消息主题长度超过127字符，消息属性长度超过32767个字符时拒接写入。
2. 获取当前可写的CommitLog文件，在内存中，使用MappedFile对象来做映射。
3. 申请PutMessageLock，串行化存储。
4. 如果是第一次创建，已0偏移量创建第一个CommitLog
5. 获取CommitLog当前的写入位置，如果写入位置小于文件大小，分裂一个ByteBuffer，设置position为当前指针
6. 设置全局唯一的消息id，由IP+端口+偏移量组成
7. 将消息序列化，写入MappedFile中的byteBuffer。
8. 设置beginTimestamp
9. 将消息添加到MappedFile中，如果当前文件足够写，开辟一个ByteBuffer
10. 同步或者异步刷盘线程来刷盘

### Mmap和PageCache

RocketMQ的读写做法逻辑：

1. 将数据文件通过Mmap（NIO中的MappedByteBuffer实现）映射到OS的虚拟内存中
2. 当消息写入时，先写到PageCache，然后异步刷盘到磁盘
3. 当消息消费者订阅消息时，要读取的数据也通过PageCache，由于PageCache一次读取4k，所以即使数据不到1k也会预加载4k到缓存中，这样下次读取的消息也容易在PageCache中命中。加快消息消费速度

#### Mmap

传统IO和Mmap
![传统IO和Mmap](/消息中间件/RocketMQ消息持久化/传统IO和Mmap.jpg)

mmap基于OS的mmap内存映射技术，将文件直接映射到用户内存，使得对文件的操作不用再需要拷贝到PageCache，而是转化为对映射地址映射的PageCache的操作，使随机读写文件和读写内存拥有相似的速度（随机地址被映射到了内存）

注意点：mmap一次只能映射1.5-2G的文件，所以CommitLog限制1G大小。在映射完成后，会占用虚拟内存，并且释放回收后，虽然不占用物理内存了，但是还是会占用虚拟内存空间，回收比较麻烦。

#### PageCache

![PageCache](/消息中间件/RocketMQ消息持久化/PageCache逻辑图.png)

PageCache属于内核，它对于应用提升I/O效率是一个投入产出比很高的解决方案


