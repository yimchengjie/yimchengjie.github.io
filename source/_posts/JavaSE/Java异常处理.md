---
title: Java异常处理
date: 2017-11-03 18:14
categories:
  - JavaSE
tags:
  - JavaSE
  - Java异常处理
toc: true
---

#### 异常处理和错误的区别

**异常:** 异常指的是程序运行时发生的不正常的时间;异常能够被程序处理,保证程序继续运行下去;例如除数为 0,文件没有找到,输入的数字格式不对;
**错误:** 错误程序没法处理,例如内存泄漏,发生错误后,一般虚拟机会选择终止程序运行,程序员需要修改代码才能解决相关错误;

#### 运行时异常与非运行时异常

Exception 有很多子类,这些子类又可以分为两大类;即**运行时异常**和**非运行时异常**
**运行时异常:** 也称为非检测异常,这些异常在编译期不检测,程序中可以选择处理,也可以不处理,如果不处理运行时会中断,但编译没问题.
**非运行时异常:** 也称为检测异常,是必须进行处理的异常,如果不处理,将发生编译期错误.

#### 异常处理的标准流程

1. **抛出异常**
   运行时异常 JVM 自行抛出,非运行时异常使用`throw`抛出
2. **捕获异常**
   `catch`语句捕获异常
3. 如**捕获成功**,异常被处理,程序继续运行
   `catch`的异常类型与抛出的异常类型匹配时
4. 如**捕获失败**,异常未被处理,程序中断运行
   `catch`的异常类型与抛出的异常类型不匹配

#### 常见的异常类型

- **Exception**: 异常层次结构的父类
- **ArithmeticException**:算术错误情况,如以 0 作除数
- **ArrayIndexOutOfBoundsException**: 数组下标越界
- **NullPointerException**: 尝试访问 null 对象成员
- **ClassNotFoundException**: 不能加载所需的类
- **ClassCastException**:对象强制类型转换出错
- **NumberFormatException**: 数字格式转换异常,如把"abc"转成数字了

#### Try-catch 代码块

```java
try {
  // 代码段 1
  // 产生异常的代码段 2
  // 代码段 3
} catch (异常类型1 e) {
  // 对异常进行处理的代码段4
} catch (异常类型2 e) {
  // 对异常进行处理的代码段5
} finally {
  // 无论是否发生异常,代码总能执行
}
```

#### Throw,Throws 关键字

- **throw**: 抛出异常(一般用于代码块和方法中)
- **throws**: 声明异常(一般用于方法中)
  
  ```java
  public void setAge(int age)throws Exception{
    if(age<=0||age>100){
      throw new Exception(); //处理方法 try-catch
    }else{
      this.age=age;
    }
  }
  ```

#### 自定义异常

```java
public class AgeException extends Exception(){
  public AgeException(){}
  public AgeException(String msg){
    super(msg);
  }
}
```

#### 使用异常机制的技巧

1. 异常处理不能代替简单的测试。 
   相比于if...else... 异常捕获的耗时是非常大的
2. 不要过分地细化异常
3. 利用异常层次结构，使用更精准的异常类
4. 不要压制异常
5. 在检测错误时，“苛刻”要比放任更好
6. 不要羞于传递异常

#### 断言

断言机制允许在测试期间向代码中插入一些检查语句， 当代码发布时，这些插入的检查语句会被自动移除。

assert 5<3==false "断言信息"

#### 记录日志

记录日志API的优点

1. 可以很容易的取消全部日志记录，或者只取消某几个级别的日志，而且打开和取消日志的操作也很简单
2. 可以很简单的禁止日志的输出
3. 日志记录可以被定向到不同的处理器， 可以用于在控制台显示， 也可以存储到文件中
4. 日志记录器和处理器都可以对记录进行过滤， 过滤器可以根据过滤实现器制定的标准丢弃无用的记录
5. 日志记录可以采用不同的方式格式化， 比如纯文本或XML
6. 应用程序可以使用多个日志记录器
7. 在默认情况， 日志系统的配置又配置文件控制

##### 基本日志

可以通过全局日志记录器， 并调用info方法
`Logger.getGlobal().info("message")`

在main中调用`Logger.getGlobal().setlevel(level.OFF)`，会取消所有日志

##### 高级日志

在一个专业应用中， 不要将所有日志都记录在一个全局日志记录器， 而是可以自定义日志记录器
`private static final Logger logger = Logger.getLogger("com.xxx.yyy");`
日志记录器也有类似包的层级结构， 此外子日志记录器还会继承父的某些属性，如日志级别， 默认的日志记录器只记录前三个

7个日志级别
- SEVERE
- WARING
- INFO
- CONFIG
- FINE
- FINER
- FINEST

##### 调试技巧

1. 可以用打印或者日志记录，记录下任意变量的值
2. 一个不太为人所知但却非常有效的技巧是在每个类中放置一个单独的main方法， 这样可以对每一个类进行单元测试
3. JUnit是一个常见的单元测试框架
4. 日志代理，利用匿名子类创建一个代理类
5. 利用printStackTrace方法，可以从任意一个异常对象中获得堆栈情况
6. 日志可以显示在控制台， 也可以用printStackTrace(PrintWriter s)方法将它发送到一个文件中
7. 让非捕获异常的堆栈轨迹出现在System.err中不是一个理想的办法，比较好的方法是将这些内容记录到一个文件中。可以调用Thread.setDefaultUnchaughtExceptionHandler方法改变非捕获异常的处理器
8. 想要观察类的加载过程，可以用-verbose标识启动Java虚拟机
9. Java虚拟机支持对Java应用程序进行监控和管理。JDK加载了一个jconsole的图形工具，用于显示内存消耗、线程使用、类加载等信息
10. 使用jmap工具获取一个堆的转储，它显示了堆的每个对象
11. 如果使用-Xprof标志运行Java虚拟机，就会运行一个基本的剖析器来跟踪那些代码中经常被调用的方法