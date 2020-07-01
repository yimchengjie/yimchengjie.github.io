---
title: Java中equals的注意事项
date: 2018-04-13 10:25:33
categories:
  - JavaSE
tags:
  - JavaSE
---

#### Java 中 equals 的注意事项

对象的 equals 方法容易抛出空指针的异常,应尽量使用常量或者有确定值的对象来调用 equals 方法
例如:

```java
String str = null;
if (str.equals("English")) {
  ...
} else {
  ..
}
```

这样容易报空指针,应该采用以下写法

```java
"English".equals(str);
```

在 jdk7 中,有一个新的工具类`java.util.Objects` 更加推荐

```java
Objects.equals(str,"English");
```
