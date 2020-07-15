---
title: Java文件
date: 2020-07-13 12:19
categories:
  - JavaSE
tags:
  - JavaSE
toc: true
---
## Java文件

java.nio.file库将Java文件操作带到了与其他编程语言相同的水平。与Java8新增的stream结合，使得文件操作变得更加优雅。

文件操作的两个基本组件：
1. 文件或者目录路径；
2. 文件本身。

----------------

### 文件和目录路径

Path对象表示一个文件或者目录路径。是一个跨操作系统和文件系统的抽象。 目的是在构造路径时不必关注底层操作系统， 代码可以在不进行修改的情况下运行在不同操作系统。

java.nio.file.Path类包含了一个重载方法 static get()， 该方法接受一个String字符串或者一个URI作为参数。并返回一个Path对象。

#### 路径分析

使用Files工具类包含一些列完整的方法用于获取Path相关的信息。

#### Paths的增减修改

我们可以通过对Path对象增加或者删除一部分，来构造一个新的Path对象。
我们使用relativize()移除Path的根路径，使用resolve()添加Path的尾路径。

### 文件系统

我们使用静态的FileSystems工具类，获取“默认”的文件系统，也可以通过Path的getFileSystem()方法来获取该Path的文件系统

### 路径监听

通过FileSystem对象，可以生成WatchService对象
WatchService可以设置一个进程对目录中的更改作出响应。

### 文件查找

通过FileSystem对象，调用getPathMatcher()可以获得一个PathMatcher。然后传入搜索模式和参数

### 文件读写

如果一个文件运行的足够快且占用的内存很小， 那么java.nio
.file.Files类中的方法可以帮助你轻松的读写文本和二进制文件。

Files.readAllLines()一次读取整个文件，产生一个`List<String>`

Files.lines()可以方便的将文件转换为行`Stream<String>`
