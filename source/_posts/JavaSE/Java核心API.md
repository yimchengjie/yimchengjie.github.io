---
title: Java核心API
date: 2017-11-10 10:09
categories:
  - JavaSE
tags:
  - JavaSE
toc: true
---

### Object

- Object 类是 Java 语言中所有类的根,所有的类都直接或间接的继承了 Object 类;
- **数组**也继承了 Object 类;
- Object 类中定义了`equals(Object obj)`方法,用来比较两个对象的虚拟地址,如果虚拟地址相同则返回 true,否则返回 false;
  - Object 类中的`equals()`方法的作用,与`==`相同,都是比较两个对象的虚地址
  - 很多类覆盖了`equals`方法,用来比较两个对象的属性值,如果属性值相同,则认为两个对象相等,例如[String 类就覆盖了 equals 方法,用来比较两个字符串的字符序列值]
- Object 类中定义了`hashCode`方法`public int hashCode()`，用来返回对象的哈希码；
  - `hashCode`方法主要为了配合基于哈希的集合类一起工作，例如 HashSet、HashMap 等；
  - 默认情况下(即没有重新 hashCode 方法时)，当两个引用的虚地址相同时，hashCode 返回相同的值，否则返回不同的值；
  - 事实上，基于哈希的集合在使用 hashCode 的时候，基本都是和 equals 一起使用；先用 hashCode 初步比较,再用 equals 比较
  - **注意**:使用的时候一起使用,重写的时候也要一起重写
- Object 类中定义了`toString`方法

  - 字符串类型是编程时最常用的类型，Object 类中定义了 toString 方法`public String toString()`，可以把任意类型对象转换成字符串返回；
  - 默认情况（没有重写 Object 类中的 toString 方法）下，返回字符串的格式为：对象类型@对象调用 hashCode 方法的返回值；
  - 返回 Object 类中默认格式的字符串几乎没有实用意义，因此很多时候，都会重写一些实体类的 toString 方法，返回需要的字符串格式；

- Object 类中定义了克隆方法`clone`
  - `Clone`方法能够“复制”一个对象，生成一个新的引用，分配新的内存空间；
  - 一个类必须实现 Cloneable 接口，才能被克隆，否则抛出异常；
  - 克隆是生成了一个新的对象，然而，对象的属性如果有引用类型，实际上还是公用；
  - 深克隆时，属性不仅值相同，同时又都存储在完全不同的内存中

### String

String 类中定义了一系列字符串相关方法，可以根据 API 文档进行学习，练习

1. 子串截取方法:
   1. `String substring(int beginIndex)`
   2. `String substring(int beginIndex,int endIndex)`
2. 检索相关方法:
   1. `int indexOf(int ch)`
   2. `int indexOf(int ch,int fromIndex)`
   3. `int indexOf(String str)`
   4. `int indexOf(String str,int fromIndex)`
   5. `int lastIndexOf(int ch)`
   6. `int lastIndexOf(int ch,int fromIndex)`
   7. `int lastIndexOf(String str)`
   8. `int lastIndexOf(String str,int fromIndex)`
   9. `char charAt(int index)`
3. 类型转换相关的方法:
   1. `static String valueOf(boolen b)`
   2. `static String valueOf(char c)`
   3. `static String valueOf(char[] data)`
   4. `static String valueOf(char[] data,int offset,int count)`
   5. `static String valueOf(double d)`
   6. `static String valueOf(float f)`
   7. `static String valueOf(int i)`
   8. `static String valueOf(long l)`
   9. `static String valueOf(Object obj)`
4. 其他方法:
   1. `int compareTo(String anotherString)`
   2. `boolean endsWith(String suffix)`
   3. `byte[] getBytes()`
   4. `byte[] getBytes(Charset charset)`
   5. `int length()`
   6. `boolean startsWith(String prefix)`
   7. `boolean startsWith(String prefix,int toffset)`
   8. `String trim()`

### 正则表达式

正则表达式就是用来描述字符串逻辑规则的工具

- 正则表达式本身也是个字符串，不过这些字符串是使用系列“元字符”组成；
- 所谓“元字符”就是预先定义的，有特殊意义的字符；例如\d 用来匹配一个数字； \w 用来匹配字母或数字或下划线或汉字等；
- 很多语言多对正则表达式提供了支持，例如 JavaScript、Java 等；
- 不同语言中使用正则表达式时，正则表达式的具体编写规则会有些小的差别，但是大体相同；

#### 正大表达式在 Java 中的使用

```java
String regex="^((13[0-9])|(15[0-3,5-9])|(18[0,2,3,5-9])|(17[0-8])|(147))\\d{8}$";
//检验的手机号码
String string="15123569087";

//第一种方式
//将正则表达式编译成Pattern对象
Pattern pattern=Pattern.compile(regex);
//使用Pattern对象为每个手机号码产生一个匹配器
Matcher matcher=pattern.matcher( string);
boolean flag=matcher.matches();
System.out.println(flag==true?"手机号正确":"手机号错误");

//第二种方式
//将正则表法式编译成Pattern对象
Pattern pattern2=Pattern.compile(regex);
boolean flag2=pattern2.matches(regex, string);
System.out.println(flag2==true?"手机号正确":"手机号错误");

//第三种方式
boolean flag3=string.matches(regex);
System.out.println(flag3==true?"手机号正确":"手机号错误");
```

#### 对象的自然比较

##### 内部比较器

- 一个类如果想【支持排序】，那么就必须实现接口`Comparable<T>`，该接口被称为对象的内部比较器；
- 该接口中只有一个方法；`int compareTo(T o)`

##### 外部比较器

- 一个类实现 Comparable 这个内部比较器后，该类支持排序，然而只能有一种排序逻辑，比较受限制；
- 可以使用外部比较器 Comparator，灵活为类定义多种比较器，此时类本身不需要实现 Comparable 接口；
- Comparator 接口中有两个方法`int compareTo(T o1,T o2)`和`boolean equals(Object obj)`

##### 对象数组的排序

- `java.util.Arrays`类是一个针对数组进行操作的工具类，其中提供了对对象数组进行排序的方法；
- 两个常用的对象数组排序方法如下：
  `static void sort(Object[] a)`
  `static <T>void sort(T[] a,ComparaTor<? super T?> c)`

### 数学 API

Math 类提供的数学运算方法:

- Math 类位于 java.lang 包中，是一个 final 类，不能被继承；
- Math 类中所有方法都是 static 方法，可以直接使用类名 Math 调用；
- Math 中定义了大量与数学运算有关的方法，包括求绝对值、三角函数、平方根等；
- Math 类是 final 类，不能被继承，所有方法都是 static 方法，可以直接用类名调用；
- Math 中的 round 方法是四舍五入，ceil 是返回大于参数且最接近参数的整数，floor 是返回小于参数且最接近参数的整数；
- Math 中的 random 方法返回[0.0,1.0)范围的值；
- Math 类中还定义了很多数学计算方法；
- Java 中的大整数 API: + Java 中整数最大范围是 long 型，64 位，如果需要使用超过 long 范围的大整数，可以使用 BigInteger 类； + BigInteger 位于 java.math 包中，定义了一系列的数学运算方法，调用这些方法可以进行计算，不能使用运算符计算； + java.math 包中还有一个类叫 BigDecimal，虽然和整数无关，我们也在此一起学习； + BigDecimal 是用来针对浮点型进行精确运算的； + BigInteger 用来对超过 long 范围整数进行运算； + BigDecimal 用来对 double、float 类型进行精确计算；

### Java 中的随机 API

- Math 类中的 random 方法可以产生随机数，然而，该方法只能生成[0.0,1.0)范围的 double 值；很多时候，可能需要生成不同类型不同范围的随机值；
- java.util 包中的 Random 类可以用来生成不同类型的随机值，功能更为强大；
- Random 类有两个构造方法，无参的构造方法创建对象后，每次都生成不同的随机数；有参的构造方法创建对象后，如果种子参数值一样，那么每次生成的随机数也相同；
- Random 类功能强大，能生成 int,float,double,boolean 各种类型的随机数；
- `random.nextInt`生成不定范围的 int 随机数，而带参数的 nextInt 生成的随机数有范围；

### UUID

- UUID 指的是通用唯一识别码，常用于分布式系统；
- 有多种生成 UUID 的策略，包括基于时间、基于名字、随机等；
- Java API 中定义了 java.util.UUID 类，对 UUID 的生成提供了支持；

### DateAPI

- java.util.Date 类表示时间，不过由于对国际化支持有限，所以 JDK1.1 之后推荐使用 java.util.Calendar 类；
- JDK1.1 版本开始，增加 Calendar 类，建议使用 Calendar 类代替 Date 类；
- Calendar 是抽象类，不能直接使用 new 创建对象；
  - Calendar 类中定义了获得实例的方法`getInstance()`,得到的实际是子类**GregorianCalendar**的对象!!
- 获得日历对象后，可以为该对象的年、月、日、时、分、秒等进行赋值：
- 实际编程中，往往需要对时间用不同的格式进行展示;
  - **SimpleDateFormat**中定义了对时间进行格式化的方法；该类继承了抽象父类 DateFormat，某些方法在父类中定义，查阅 API 文档时注意；
  - 可以自定义一个模式字符串来构建 SimpleDateFormat 对象：
  - 通常使用 format 方法进行格式化；

### JDK8 中的新 API

- JDK8 中定义了`java.time.LocalDate`，用来表示日期，默认格式是 yyyy-MM-dd；该类不包含时间信息；
