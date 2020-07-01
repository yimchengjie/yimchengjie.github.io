---
title: Lock锁
categories:
  - Java并发编程
tags:
  - Java并发编程
  - Java锁
toc: true
date: 2019-03-17 18:17:53
---

### Lock

Lock 是一个锁的接口， 提供获取锁和解锁的方法（lock，trylock，unlock）
是一种用代码实现的锁
**注意:**当 Lock 没有指向 unlock 释放锁的时候,即为可重入锁,其他线程在锁没有释放的时候还是无法获取锁,即使没有线程在使用锁, 但还是有一个线程在占用锁
**ReentrantLock**是 Lock 接口的一个实现类， 它实现了 Lock 中的方法， 但是使用 Lock 的时候必须注意它不会像 synchronized 执行完后或者抛出异常后自动释放锁， 而是需要你主动释放锁， 所以我们必须在使用 Lock 时加上 try{}catch{}finally{}块，并且在 finally 中释放占用的锁资源。

使用 Lock 和 synchronized 最大的区别就是当**使用 synchroniized 时一个线程抢占资源，其他线程必须等待**，而**使用 Lock，一个线程抢占到锁资源，其他的线程可以不等待或者设置等待时间**， 实在抢不到可以去做其他的业务逻辑

#### **ReadWriteLock 读写锁**

它可以实现读写锁， 当读取的时候线程会获得 read 锁， 其他线程也可以获取 read 锁，同时并发的去读取，但是写程序运行获取到 write 锁时， 其他线程是不能进行操作的，因为 write 是排它锁，而上面介绍的两种 Lock 和 synchronized 不管是 read 还是 write 没有抢到锁的线程都会被阻塞或者中断，ReadWriteLock 也是个接口，里面定义了两种方法 readLock()和 writeLock()，他的一个实现类是 ReentrantReadWriteLock 是 ReadWriteLock 的实现类
**注意**读锁被占用的时候,写锁不能被占用

### **小记:**

<u>Lock 比内置锁更具灵活性,可以设置方法不去等待锁,做其他的逻辑操作. 它具备和内置锁同样的内存语义</u>
