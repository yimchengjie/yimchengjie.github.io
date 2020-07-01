---
title: Java线程间通信
categories:
  - Java并发编程
tags:
  - Java并发编程
toc: true
date: 2020-05-31 16:03:31
---
## Java线程间通信

### synchronized同步关键字

使用synchronized关键字,同步线程,可以让线程按照一定顺序去执行;

synchronized关键字,线程需要不断尝试获取锁,这会耗费CPU资源

### 等待/通知-wait()/notify()

Java线程的等待/通知机制,是使用Object类的wait()方法和notify(),notifyAll()方法;

当一个线程A获取锁,并开始执行,业务执行完成后调用notifyAll()和wait()方法,使自己进入等待状态,并唤醒其他线程,被唤醒的线程B就会获取锁, 并执行;

wait()和notify()是针对锁对象, 而不是线程的方法, 是锁来调用wait()和notify()

### 信号量通信volatile关键字

volatile关键字保证数据的可见性, 如果一个变量使用了volatile关键字, 那么它一改变, 其他线程的内存就会立马更新成改变后的数据。

### join()方法

join()方法是Thread的一个实例方法,它的作用是使当前所在的线程等待调用join()方法的线程执行完后在执行。

比如子线程有许多耗时操作， 而主线程要获取子线程得到的数据， 那就需要让主线程去等待子线程。

### ThreadLocal类

使用ThreadLocal在线程中创建线程本地变量，内部是一个Map来维护数据，ThreadLocal可以让每个线程都拥有自己独立的ThreadLocal副本，使每个线程可以访问自己的独立副本变量。

使用时，可以利用构造方法，给多个线程传递同一个ThreadLocal实例， 多个线程会创建出独立的副本， 而不影响其他线程中的ThreadLocal。