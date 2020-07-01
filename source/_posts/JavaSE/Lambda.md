---
title: Lambda
date: 2017-11-18 13:40
categories:
  - JavaSE
tags:
  - JavaSE
  - Lambda
toc: true
---

#### 函数式接口

我们把只拥有一个方法的接口称为函数式接口.
我们并不需要额外的工作来声明一个接口是函数式接口：编译器会根据接口的结构自行判断（判断过程并非简单的对接口方法计数：一个接口可能冗余的定义了一个  Object  已经提供的方法，比如  toString()，或者定义了静态方法或默认方法，这些都不属于函数式接口方法的范畴）。不过 API 作者们可以通过  @FunctionalInterface  注解来显式指定一个接口是函数式接口（以避免无意声明了一个符合函数式标准的接口），加上这个注解之后，编译器就会验证该接口是否满足函数式接口的要求。

函数式接口的实现引入了一个全新的结构化函数类型.我们也称为是"箭头"类型.

##### Java8 中加入新的包：java.util.function

它包含了常用的函数式接口:

1. `Predicate<T>`: 接收`T`并返回`boolean`
2. `Consumer<T>`: 接收`T`,不返回值
3. `Function<T, R>`: 接收`T`,返回`R`
4. `Supplier<T>`: 提供`T`,不接收值
5. `Unaryoperator<T>`: 接收`T`,返回`T`
6. `Binary0perator<T>`: 接收两个`T`,返回`T`

#### Lambda 表达式

lambda 表达式是匿名方法，它提供了轻量级的语法，从而解决了匿名内部类带来的语法过于冗余.

- 下面是一些表达式:
  
  ```java
  (int x,int y)->x+y
  ()->42
  (String s)->System.out.println(s);
  ```
  
- lambda 表达式的语法由参数列表、箭头符号  `->`  和函数体组成。函数体既可以是一个表达式，也可以是一个语句块：

  - 表达式:表达式会被执行然后返回执行结果.
  - 语句块:语句块中的语句会被依次执行,就像方法中的语句一样
    - `return`语句会把控制权交给匿名方法的调用者
    - `break`和`continue`只能在循环中使用
    - 如果函数体有返回值,那么函数体内部的每一条路径都必须返回值

- 目标类型:
  - 编译器负责推导**lambda**表达式类型,它利用 lambda 表达式所在上下文**所期待的类型**进行推导,这个**被期待的类型**被称为==目标类型==.**lambda**表达式只能出现在目标类型为函数式接口的上下文中.
  - 当然**lambda**对于**目标类型**也是有要求的,编译器会检查 lambda 表达式的类型和目标类型的方法签名是否一致,当且仅当下面所有条件均满足时,lambda 表达式才可以被赋给目标类型`T`:
    - `T`是一个函数式接口
    - `lambda`表达式的参数和`T`的方法参数在数量和类型上一一对应
    - `lambda`表达式的返回值和`T`的方法返回值相兼容
    - `lambda`表达式内锁抛出的异常和`T`的方法`throws`类型相兼容

#### Java 内置函数式接口

为了免去用户每次使用 Lamdba 表达式时,都自行创建函数式接口,java 中提供了四大核心内置函数式接口:

```java
/**
    * Consumer<T> :消费型接口
    *          void accept(T t);
    *
    * Supplier<T> :供给型接口
    *          T get();
    *
    * Function<T,R> :函数型接口
    *          R apply(T t);
    *
    * Predicate<T> :断言型接口
    *          boolean test(T t);
    */
public class TestLambda3 {

    //Consumer<T> 消费型接口：
    public void happy(double money,Consumer<Double> con){
        con.accept(money);
    }
    @Test
    public void test1(){
        happy(1000,(m) ->System.out.println("消费："+m+"元"));
    }

    //Supplier<T> 供给型接口:
    //需求：产生指定个数的整数，并放入集合中
    public List<Integer> getNumList(int num,Supplier<Integer> sup){
        List<Integer> list=new ArrayList<>();
        for (int i = 0; i < num; i++) {
            Integer n=sup.get();
            list.add(n);
        }
        return list;
    }
    @Test
    public void test2(){
        List<Integer> numList=getNumList(10, ()->(int)(Math.random()*100));
        for (Integer num : numList) {
            System.out.println(num);
        }
    }

    //Function<T,R> 函数型接口:
    //需求：处理字符串
    public String strHandler(String str,Function<String,String> fun){
        return fun.apply(str);
    }
    @Test
    public void test3(){
        String newStr=strHandler("\t\t\t 哈哈哈  ", (str)->str.trim());
        System.out.println(newStr);
        String subStr=strHandler("abcdef", (str)->str.substring(2,4));
        System.out.println(subStr);
    }

    //Predicate<T> 断言型接口：
    //需求：将满足条件的字符串，放入集合中
    public List<String> filterStr(List<String> list,Predicate<String> pre){
        List<String> strList=new ArrayList<>();
        for ( String str : list) {
            if(pre.test(str)){
                strList.add(str);
            }
        }
        return strList;
    }
    @Test
    public void test4(){
        List<String> list=Arrays.asList("Hello","jj","Lambda","www","ok");
        List<String> strList=filterStr(list, (s)->s.length()>3);
        for (String string : strList) {
            System.out.println(string);
        }
    }
}
```
