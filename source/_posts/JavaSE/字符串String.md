---
title: 字符串String
date: 2020-07-12 10:19
categories:
  - JavaSE
tags:
  - JavaSE
toc: true
---
## 字符串String

----------

### 不可变的String

String对象是不可变的， String类中的每一个看起来会修改String值的方法，实际上都会创建一个全新的String对象， 以包括修改后的字符串内容。 而最初的String对象则丝毫未动。

### “+”的重载与StringBuilder

String的对象不可变，你可以添加任意多的引用指向它， 因为String是只读的， 任何引用都不可能修改它的值。

不可变性带来一定的效率问题，“+”操作符的重载使用了StringBuilder类

编译器自动创建一个StringBuilder对象， 用于构建最终的String， 并且每个字符串调用一次append()方法， 最后toString()生成结果。

但是编译器自动优化不是万能的， 当需要用到循环时，最好自己创建一个StringBuilder对象

### 打印内存地址

当你想要调用toString()方法打印对象地址时， 不要在方法中使用this，容易造成意外递归。

应该使用super.toString()方法



