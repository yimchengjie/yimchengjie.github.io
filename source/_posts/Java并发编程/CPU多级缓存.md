---
title: CPU多级缓存
categories:
  - Java并发编程
tags:
  - cache
  - 内存屏障
toc: true
date: 2019-01-15 18:31:16
---

## CPU 多级缓存

![CPU多级缓存](CPU多级缓存.png)
![为何使用cpu_cache](为何使用cpu_cache.png)
![cpu_cache意义](cpu_cache意义.png)

### cache 带来的问题

cache 给系统带来性能上飞跃的同时，也引入了新的问题“**缓存一致性问题**”。设想如下场景（cpu 一共有两个核，core1 和 core2）： 以 i++为例，i 的初始值是 0.那么在开始每个核都存储了 i 的值 0，当第 core1 块做 i++的时候，其缓存中的值变成了 1，即使马上回写到主内存，那么在回写之后 core2 缓存中的 i 值依然是 0，其执行 i++，回写到内存就会覆盖第一块内核的操作，使得最终的结果是 1，而不是预期中的 2。

缓存一致性
为了达到数据访问的一致，需要各个处理器在访问缓存时遵循一些协议，在读写时根据协议来操作，常见的协议有 MSI，MESI，MOSI 等。我们介绍其中最经典的 MESI 协议。
在 MESI 协议中，每个 cache line 有 4 个状态，可用 2 个 bit 表示，它们分别是：
|状态| 描述|
|:--:|:--:|
|M(Modified)| 这行数据有效，数据被修改了，和内存中的数据不一致，数据只存在于本 Cache 中。|
|E(Exclusive)| 这行数据有效，数据和内存中的数据一致，数据只存在于本 Cache 中。|
|S(Shared)| 这行数据有效，数据和内存中的数据一致，数据存在于很多 Cache 中。|
|I(Invalid)| 这行数据无效|

![cpu读取事件](cpu读取事件.png)

<table>
  <tr>
    <th>当前状态</th>
    <th>事件</th>
    <th>行为</th>
    <th>下一状态</th>
  </tr>
  <tr>
    <td rowspan="4">I(Invalid)</td>
    <td>Local Read</td>
    <td>如果其它Cache没有这份数据，本Cache从内存中取数据，Cache line状态变成E； 如果其它Cache有这份数据，且状态为M，则将数据更新到内存，本Cache再从内存中取数据，2个Cache 的Cache line状态都变成S； 如果其它Cache有这份数据，且状态为S或者E，本Cache从内存中取数据，这些Cache 的Cache line状态都变成S</td>
    <td>E/S</td>
  </tr>
  <tr>
    <td>Local Write</td>
    <td>从内存中取数据，在Cache中修改，状态变成M； 如果其它Cache有这份数据，且状态为M，则要先将数据更新到内存； 如果其它Cache有这份数据，则其它Cache的Cache line状态变成I</td>
    <td>M</td>
  </tr>
  <tr>
    <td>Remote Read</td>
    <td>既然是Invalid，别的核的操作与它无关</td>
    <td>I</td>
  </tr>
  <tr>
    <td>Remote Write</td>
    <td>既然是Invalid，别的核的操作与它无关</td>
    <td>I</td>
  </tr>

  <tr>
    <td rowspan="4">E(Exclusive)</td>
    <td>Local Read</td>
    <td>从Cache中取数据，状态不变</td>
    <td>E</td>
  </tr>
  <tr>
    <td>Local Write</td>
    <td>修改Cache中的数据，状态变成M</td>
    <td>M</td>
  </tr>
  <tr>
    <td>Remote Read</td>
    <td>数据和其它核共用，状态变成了S</td>
    <td>S</td>
  </tr>
  <tr>
    <td>Remote Write</td>
    <td>数据被修改，本Cache line不能再使用，状态变成I</td>
    <td>I</td>
  </tr>

  <tr>
    <td rowspan="4">S(Shared)</td>
    <td>Local Read</td>
    <td>从Cache中取数据，状态不变</td>
    <td>S</td>
  </tr>
  <tr>
    <td>Local Write</td>
    <td>修改Cache中的数据，状态变成M， 其它核共享的Cache line状态变成I</td>
    <td>M</td>
  </tr>
  <tr>
    <td>Remote Read</td>
    <td>状态不变</td>
    <td>S</td>
  </tr>
  <tr>
    <td>Remote Write</td>
    <td>数据被修改，本Cache line不能再使用，状态变成I</td>
    <td>I</td>
  </tr>
  <tr>
    <td rowspan="4">M(Modified)</td>
    <td>Local Read</td>
    <td>从Cache中取数据，状态不变</td>
    <td>M</td>
  </tr>
 <tr>
    <td>Local Write</td>
    <td>修改Cache中的数据，状态不变</td>
    <td>M</td>
  </tr>
  <tr>
    <td>Remote Read</td>
    <td>这行数据被写到内存中，使其它核能使用到最新的数据，状态变成S</td>
    <td>S</td>
  </tr>
  <tr>
    <td>Remote Write</td>
    <td>这行数据被写到内存中，使其它核能使用到最新的数据，由于其它核会修改这行数据， 状态变成I</td>
    <td>I</td>
  </tr>
</table>

### 指令重排序

![cpu乱序优化](cpu乱序优化.png)
指令重排场景:当 CPU 写缓存时发现缓存区块正被其他 CPU 占用,为了提高 CPU 处理性能,可能将后面的读缓存命令优先执行，那么问题来了，那些指令不是在所有场景下都能进行重排，除了本身的一些规则之外，我们还需要确保多 CPU 的高速缓存中的数据与内存保持一致性，不能确保内存与 CPU 缓存数据一致性的指令也不能重排，内存屏障正式通过阻止屏障两边的指令重排序来避
免编译器和硬件的不正确优化而提出的一种解决办法。

#### 内存屏障

处理器提供了两个内存屏障指令用于解决以上问题

1. 写内存屏障(Store Memory Barrier)
   能让写入缓存中的最新数据更新写入主内存, 让其他线程可见
2. 读内存屏障(Load Memory Barrier)
   让高速缓存中的数据失效,强制重新从主内存加载数据

#### volatile

JAVA 中的 volatile 关键字正是使用了内存屏障。如果字段是 volatile，java 内存模型将在写操作后插入一个写屏障指令，在读操作前插入一个读屏障指令。这意味着，如果你对一个 volatile 字段进行写操作，你必须知道：

1. 一旦你完成写入，任何访问这个字段的线程将会得到最新的值。
2. 在你写入前，会保证所有之前发生的事已发生，并且任何更像过的数据值也是可见的。因为内存屏障会把之前的写入值都刷新到缓存。

**注意**: 内存屏障会导致不可以尽可能地高校利用 CPU，另外刷新缓存亦会有开销。所以不要以为用 volatile 代替锁操作就一点事都没有。
