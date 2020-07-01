---
title: SteamAPI
date: 2017-11-25 11:30
categories:
  - JavaSE
tags:
  - JavaSE
toc: true
---

### Stream API

Stream 位于包 java.util.stream .\* 是 Java8 中处理集合的关键抽象概念，它可以指定你希望对集合进行的操作，可以执行非常复杂的查找、过滤和映射数据等操作。使用 Stream API 对集合数据进行操作，就类似于使用 SQL 执行的数据库查询。也可以使用 Stream API 来并行执行操作。简而言之，Stream API 提供了一种高效且易于使用的处理数据的方式。

#### 流(Stream)到底是什么呢

是数据渠道，用于操作数据源（集合、数组等）所生成的元素序列。
**集合讲的是数据，流讲的是计算！**

##### 注意

1. Stream 自己不会存储元素。
2. Stream 不会改变源对象。相反，他们会返回一个持有结果的新 Stream。
3. Stream 操作是延迟执行的。这意味着他们会等到需要结果的时候才执行。

#### Stream 的操作三步骤

1. 创建 Stream
   一个数据源（如：集合、数组），获取一个流
2. 中间操作
   一个中间操作链，对数据源的数据进行处理
3. 终止操作（终端操作）
   一个终止操作，执行中间操作链，并产生结果

##### 1. 创建 Stream

1. 可以通过`Collection`系列集合提供的`stream()`或`parallelStream()`方法
   `default Stream< E> stream()` : 返回一个顺序流
   `default Stream< E> parallelStream()` : 返回一个并行流
2. 通过 `Arrays` 中的静态方法`stream()`获取数组流
   `static <T> Stream<T> stream( T[] array)`: 返回一个流
   重载形式，能够处理对应基本类型的数组：
   `public static IntStream stream(int[] array)`
   `public static LongStream stream(long[] array)`
   `public static DoubleStream stream(double[] array)`
3. 通过 Stream 类中的静态方法 of()，通过显示值创建一个流。它可以接收任意数量的参数。
   `public static< T> Stream< T> of(T… values)` : 返回一个流
4. 创建无限流
   可以使用静态方法 `Stream.iterate()` 和`Stream.generate()`, 创建无限流。
   迭代 `public static< T> Stream< T> iterate(final T seed, final UnaryOperator< T> f)`
   生成 `public static< T> Stream< T> generate(Supplier< T> s)`
   `java //创建Stream @Test public void test1(){ //1.可以通过Collection 系列集合提供的stream()或parallelStream() List<String> list = new ArrayList<>(); Stream<String> stream1 = list.stream(); //2.通过 Arrays 中的静态方法stream()获取数组流 Employee[] emps=new Employee[10]; Stream<Employee> stream2=Arrays.stream(emps); //3.通过Stream 类中的静态方法of() Stream<String> stream3=Stream.of("aa","bb","cc"); //4.创建无限流 //迭代 Stream<Integer> stream4=Stream.iterate(0, (x) -> x+2); stream4.limit(10).forEach(System.out::println); //生成 Stream.generate(() -> Math.random()).limit(5).forEach(System.out::println); }`

#### 2.中间操作

多个中间操作可以连接起来形成一个流水线，除非流水线上触发终止操作，否则中间操作不会执行任何的处理！而在终止操作时一次性处理，成为“惰性求值”。

1. 筛选与切片  
    `filter(Predicate p)`: 接收 lambda,从流中排除某些元素
    `distinct()`: 筛选,通过流所生成元素的`hashCode()`和`equals`去除重复元素
    `limit(long maxSize)`: 截断流,使元素不超过给定数量
    `skip(long n)`: 跳过元素,返回一个扔掉了前 n 个元素的流,若流中元素不足 n 个,则返回一个空流,与`limit(n)`互补

    ```java
    //中间操作

        List<Employee> employees=Arrays.asList(
                new Employee("张三",18,9999.99),
                new Employee("李四",58,5555.55),
                new Employee("王五",26,3333.33),
                new Employee("赵六",36,6666.66),
                new Employee("田七",12,8888.88),
                new Employee("田七",12,8888.88)
                );

        /* 筛选与切片
        *  filter--接收Lambda，从流中排除某些元素。
        *  limit--截断流，使其元素不超过给定数量。
        *  skip(n)--跳过元素，返回一个扔掉了前n个元素的流。若流中元素不足n个，则返回一个空流。与limit(n) 互补
        *  distinct--筛选，通过流所生成元素的 hashCode() 和 equals() 去掉重复元素
        */

        //内部迭代：迭代操作由 Stream API 完成
        @Test
        public void test1(){
            //中间操作：不会执行任何操作
            Stream<Employee> stream=employees.stream().filter((e) -> e.getAge()>35 );
            //终止操作：一次性执行全部内容，即 惰性求值
            stream.forEach(System.out::println);
        }
        //外部迭代
        @Test
        public void test2(){
            Iterator<Employee> it=employees.iterator();
            while(it.hasNext()){
                System.out.println(it.next());
            }
        }

        @Test
        public void test3(){//发现“短路”只输出了两次，说明只要找到 2 个 符合条件的就不再继续迭代
            employees.stream().filter((e)->{
                System.out.println("短路！");
                return e.getSalary()>5000;
            }).limit(2).forEach(System.out::println);
        }

        @Test
        public void test4(){
            employees.stream().filter((e)->e.getSalary()>5000).skip(2)//跳过前两个
            .distinct()//去重，注意：需要Employee重写hashCode 和 equals 方法
            .forEach(System.out::println);
        }
        ```

2. 映射

    ```java
    /* 映射
    * map--接收Lambda，将元素转换成其他形式或提取信息。接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新元素。
    * flatMap--接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流
    */
    @Test
    public void test5(){
        List<String> list=Arrays.asList("aaa","bbb","ccc","ddd");
        list.stream() .map((str)->str.toUpperCase())
            .forEach(System.out::println);
        System.out.println("------------------------");
        employees.stream().map(Employee::getName)
                    .forEach(System.out::println);
        System.out.println("------------------------"); Stream<Character> sm=list.stream()
        .flatMap(TestStream::filterChatacter);
        sm.forEach(System.out::println);
    }
    public static Stream<Character> filterChatacter(String str){
        List<Character> list=new ArrayList<>();
        for (Character ch : str.toCharArray()) {
            list.add(ch);
        }
        return list.stream();
    }
    ```

3. 排序

    ```java
    /*排序
    * sorted()-自然排序（按照对象类实现Comparable接口的compareTo()方法 排序）
    * sorted(Comparator com)-定制排序（Comparator）
    */
    @Test
    public void test7(){
        List<String> list=Arrays.asList("ccc","bbb","aaa");
        list.stream().sorted().forEach(System.out::println);
        System.out.println("------------------------");
        employees.stream().sorted((e1,e2)->{
            if(e1.getAge().equals(e2.getAge())){
                return e1.getName().compareTo(e2.getName());
            }else{
                return e1.getAge().compareTo(e2.getAge());
            }
        }).forEach(System.out::println);
    }
    ```

#### 3. 终止操作

终止操作会从流水线生成结果。其结果可以是任何不是流的值，例如：List、Integer，甚至是 void。

1. 查找与匹配

   ```java
   /*
   * 查找与匹配
   */
   @Test
   public void test1(){
       //allMatch-检查是否匹配所有元素
       boolean b1=employees.stream().allMatch((e)->e.getStatus().equals(Status.BUSY));
       System.out.println(b1);//false
       boolean b2=employees.stream().anyMatch((e)->e.getStatus().equals(Status.BUSY));
       System.out.println(b2);//true
       //noneMatch-检查是否没有匹配所有元素
       booleanb3=employees.stream().noneMatch((e)->e.getStatus().equals(Status.BUSY));
       System.out.println(b3);//false
       //findFirst-返回第一个元素//Optional是Java8中避免空指针异常的容器类
       Optional<Employee> op=employees.stream().sorted((e1,e2)->Double.compare(e1.getSalary(), e2.getSalary())).findFirst();
       System.out.println(op.get());
       //findAny-返回当前流中的任意元素
       Optional<Employee> op2=employees.parallelStream().filter((e)->e.getStatus().equals(Status.FREE)).findAny();
       System.out.println(op2.get());
       //count-返回流中元素的总个数
       Long count=employees.stream().count();
       System.out.println(count);//5
       //max-返回流中最大值
       Optional<Employee> op3=employees.stream().max((e1,e2)->Double.compare(e1.getSalary(), e2.getSalary()));
       System.out.println(op3.get());
       //min返回流中最小值
       Optional<Double>op4=employees.stream().map(Employee::getSalary).min(Double::compare);
           System.out.println(op4.get());//3333.33
       }
   ```

2. 归约

   ```java
   /*归约
   * reduce(T identity,BinaryOperator b) / reduce(BinaryOperator b)-可以将流中元素反复结合起来，得到一个值。
   */
   @Test
   public void test3(){
       List<Integer> list=Arrays.asList(1,2,3,4,5,6,7,8,9,10);
       Integer sum=list.stream()
       reduce(T identity,BinaryOperator b).reduce(0, (x,y)->x+y);
       //0为起始值
       System.out.println(sum);
       System.out.println("--------------------------");
       Optional<Double> op=employees.stream().map(Employee::getSalary).reduce(Double::sum);
       System.out.println(op.get());
   }
   ```

3. 收集

   ```java
   /*
   * 收集
   * collect-将流转换为其他形式，接收一个Collector接口的实现，用于给Stream中元素做汇总的方法。
   */
   @Test
   public void test4(){
       List<String> list=employees.stream().map(Employee::getName).collect(Collectors.toList());
       list.forEach(System.out::println);
       System.out.println("----------------------------");
       Set<String> set=employees.stream().map(Employee::getName).collect(Collectors.toSet());
       set.forEach(System.out::println);
       System.out.println("----------------------------");
       //总和
       Long count=employees.stream().collect(Collectors.counting());
       System.out.println(count);
       //平均值
       Double avg=employees.stream().collect(Collectors.averagingDouble(Employee::getSalary));
       System.out.println(avg);
       //总和
       Double sum=employees.stream().collect(Collectors.summingDouble(Employee::getSalary));
       System.out.println(sum);
       //最大值
       Optional<Employee> max=employees.stream().collect(Collectors.maxBy((e1,e2)->Double.compare(e1.getSalary(), e2.getSalary())));
       System.out.println(max.get());
       //最小值
       Optional<Double> min=employees.stream().map(Employee::getSalary).collect(Collectors.minBy(Double::compare));
       System.out.println(min.get());
       System.out.println("----------------------------");
       //分组
       Map<Status,List<Employee>> map=employees.stream().collect(Collectors.groupingBy(Employee::getStatus));
       System.out.println(map);
       //分区
       Map<Boolean,List<Employee>> map3=employees.stream().collect(Collectors.partitioningBy((e)->e.getSalary()>8000));
       System.out.println(map3);
   )
   ```
