---
title: JDK内置命令工具
categories:
  - Java系统调优
tags:
  - Java
  - JVM
  - JDK
toc: true
date: 2019-06-18 14:56:05
---
## JDK内置命令工具

### javap命令

java反编译工具, 主要用于根据Java字节码文件反汇编为Java源代码文件.

### jps命令

显示当前所有java进程pid的命令

### jstat命令

监视Java虚拟机统计信息

### jcmd命令

可以替代jps工具查看本地的jvm信息

### jinfo命令

可以查看运行中的jvm的全部参数, 还可以设置部分参数

### jhat

分析java堆的命令,可以将堆中的对象以html的形式展现出来,支持对象查询语言SQL

### jmap

打印出java进程内存中对象的情况,或者将JVM中的堆,以二进制输出成文本

### jstack

用于打印出给定的java进程ID或core file或远程调试服务的堆栈信息, 如果在64位机器, 需要指定选项`-J-d64`

### Jconsole

可视化的监视管理控制台

### JvisualVM

可视化的JVM监控工具
