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

#### 断言

assert 5<3==false "断言信息"
