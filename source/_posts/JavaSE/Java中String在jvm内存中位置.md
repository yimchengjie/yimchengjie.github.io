---
title: Java中String在jvm内存中位置
date: 2020-03-04 17:14:33
categories:
  - JavaSE
tags:
  - JavaSE
  - String
  - JVM
---
## Java中String在jvm内存中位置

```java
public static void main(String[] args) {
        //intern 如果存在在常量池, 返回常量池中的,
        //       如果存在在堆, 返回堆中的对象引用
        String a="hello";  //在常量池
        String b="hello";  //在常量池
        System.out.println(a==b);  //true

        String c=new String("hello");  //在堆
        System.out.println(b==c);  //false

        String d="world"; //在常量池

        String aa=a+d;  //在常量池
        String bb=c+d;  //在堆
        System.out.println(aa==bb); //false

        bb.intern();  //将堆中引用创建到常量池
        String cc=new String("helloworld"); //创建在堆中
        System.out.println(bb==cc);    //false
        System.out.println(bb==cc.intern());  //true   cc.intern()发现了在常量池中bb创建的引用,并返回
    }
```

总结:
`String str=new String("abc");` 会在常量池和堆中都创建abc, str指向堆中的abc
`String str="abc";`只在常量池创建abc
`String str=new String("abc")+"def";`在常量池中创建abc,def;在堆中创建abc和abcdef; str指向堆中abcdef;
