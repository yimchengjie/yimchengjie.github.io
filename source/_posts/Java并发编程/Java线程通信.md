---
title: Java线程通信
categories:
  - Java并发编程
tags:
  - JavaSE
  - Java并发编程
toc: true
date: 2019-01-23 10:19:49
---

## 概述

线程与线程之间不是相互独立的个体，它们彼此之间需要相互通信和写作，最典型的例子就是生产者-消费者问题：当队列满时，生产者需要等待队列有空间才能继续往里面放入商品，而在等待的期间内，生产者必须释放对临界资源（即队列）的占用权。因为生产者如果不释放对临界资源的占用权，那么消费者就无法消费队列中的商品，就不会让队列有空间，那么生产者就会一直无限等待下去。因此一般情况下，当队列满时，会让生产者交出对临界资源的占用权，并进入挂起状态。然后等待消费者消费了商品，然后消费者通知生产者队列有空间了。同样地，当队列空时，消费者也必须等待，等待生产者通知它队列中有商品了。这种互相通信的过程就是线程间的协作。

### 线程之间的通信有以下

1. 文件共享
2. 网络共享
3. 共享变量
4. jdk 提供的线程协调 API

#### 1. 文件共享

线程 1 写入文件 a.txt,线程 2 从 a.txt 中读取数据

#### 3. 变量共享

线程 1 写入公共变量线程 2 从公共变量读取数据

#### 4. jdk 提供的线程协调 API

多线程协作的典型场景:生产者-消费者模型

示例:线程 1 去买包子,没有包子,则不再执行.线程 2 生产出包子,通知线程 1 继续执行

**jdk 已弃用的 suspend/resume 挂起和唤醒**
被弃用的主要原因是,容易写出死锁的代码,

1. 在同步代码中使用,suspend 挂起不会释放锁,产生死锁
2. suspend 一定要在 resumee 之前执行,否则

#### **wait/notify 机制**

在这之前，线程间通过共享数据来实现通信，即多个线程主动地读取一个共享数据，通过同步互斥访问机制来保证线程的安全性。等待/通知机制主要由 Object 类中的 wait()、notify()和 notifyAll()三个方法来实现，这三个方法均非 Thread 中所申明的方法，而是 Object 类中申明的方法。原因是每个对象都拥有 monitor（锁），所以让但却概念线程等待某个对象的锁，当然应该通过这个对象来操作，而不是当前线程来操作，因为当前线程可能会等待多个线程的锁，如果通过线程来操作就会非常复杂。
**wait()——让当前线程释放对象锁并进入等待（阻塞）状态**
**notify()——唤醒一个正在等待相应对象锁的线程，使其进入就绪队列，以便在当前线程释放锁后竞争锁，进而得到 CPU 的执行。**
**notifyAll()——唤醒所有正在等待相应对象锁的线程，使它们进入就绪队列，以便在当前线程释放锁后竞争锁，进而得到 CPU 的执行。**

这些方法只能由同一对象锁的持有者线程调用,也就是**必须写在同步块里**,否则会抛出 illegalMonitorStateException

wait 会自动解锁,但对顺序调用还是有需求.不能再 notify 后调用

#### 锁对象

每个锁对象都有两个队列，一个是就绪队列，一个是阻塞队列。就绪队列存储了已就绪（将要竞争锁）的线程，阻塞队列存储了被阻塞的线程。当一个阻塞线程被唤醒后，才会进入就绪队列，进而等待 CPU 的调度，反之，当一个线程被 wait 之后，就会进入阻塞队列，等待被唤醒。

#### **park/unpark 机制**

线程调用 park 则等待"许可",unpark 为指定线程提供"许可"

不要求 park 和 unpark 方法的调用顺序
多次调用 unpark,再调用 park,线程会直接运行,但不会叠加, 连续多次调用 park,只有第一次能拿到"许可"
虽然没有顺序要求,但是 park 不会释放锁

**注意:** 不要使用 if 语句来判断,是否进入等待状态,应该在循环中检测等待条件, 原因是处于等待状态的线程可能会收到错误警报和伪唤醒,如果不在循环条件中检查等待条件,程序就会在没有满足结束条件的情况下退出.
