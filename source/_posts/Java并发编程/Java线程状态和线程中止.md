---
title: Java线程状态和线程中止
categories:
  - Java并发编程
tags:
  - JavaSE
  - Java并发编程
toc: true
date: 2019-01-09 09:19:46
---

### 线程中止

1. **设置标记位**(受限与线程中业务逻辑有循环条件)
   thread.setFlag(false);
2. **使用 stop()**，不安全，已经被废弃
   thread.stop()，因为 stop()方法会解除由线程获得的所有锁，当在一个线程对象上调用 stop()方法时，这个线程对象所运行的线程会立即停止
3. **使用 Thread 类的 interrupt()** 方法中断线程
   调用 interrupt()方法只会给线程设置一个为 true 的终端标志，而设置之后，则根据线程当前状态进行不同的后续操作

### 线程的五大状态及其转换

线程的五大状态分别为:
**创建状态（new）**、**就绪状态（Runnable）**、**运行状态（Running）**、**阻塞状态（Blocked）**、**死亡状态（Dead）**。

五大状态之间的关系图：
![java线程状态](java线程状态.png)

#### （1）新建状态：即单纯的创建一个线程

**创建线程有三种方式**；
**1.集成 Thread 类创建线程**
使用集成 Thread 类创建线程时，首先需要创建一个类集成 Thread 类并覆写 Thread 类
run()方法，在 run()方法中，需要写线程要执行的任务
但是调用 run()方法并不是真正的启动一个线程，真正的启动线程，需要调用的时 Thread 类的 start()方法，而 start()方法会自动调用 run()方法，从而启动一个线程。
**说明：** main 方法其实也是一个线程，是该进程的主线程。但是在使用多线程技术时，代码的运行结果与代码调用的顺序无关，因为线程是一个子任务，CPU 以不确定的方式或者说以随机的时间来调用线程中的 run()方法，所以会出现每次运行结果不同的情况。

**2.实现 Runnable 接口创建线程**
Thread 类的核心功能就是进行线程的启动，但一个类为了实现多线程直接取继承 Thread 类时出现的问题就是：单集成的局限性！所以 Java 中还提供了另一种实现多线程的方法：实现 Runnable 接口来创建多线程。
**注意：** 启动一个线程的唯一方法就是调用 Thread 类的 start()方法，抓住这点去建立与 Thread 类之间的关系。
Runnable 接口中只有一个抽象方法就是 run()方法。那怎么使用 Runnable 接口去创建线程呢？（如何执行 start()方法呢）
**第一步：定义一个类实现 Runnable 接口的抽象方法 run()方法。** 此时 Thread 类有一个 Thread 类的构造方法 public Thread(Runnable target)方法，参数用于接收 Runnable 接口的实例化对象，所以在 Runnable 接口与 Thread 类之间就建立起了联系，从而可以调用 Thread 类的 start()方法启动一个线程。所以
**第二步：利用 Thread 类的 public Thread(Runnable target)构造方法与 Runnable 接口建立关系实例化 Thread 类的对象**；
**第三步：调用 Thread 类的 start()方法启动线程。**

**3.实现 Callable 接口创建线程**
Runnable 接口的 run()方法没有返回值，而 Callable 接口中的 call()方法有返回值，若某些线程执行完成后需要一些返回值的时候，就需要用 Callable 接口创建线程。
再次强调：**启动一个线程的唯一方法就是调用 Thread 类的 start()方法**，那么，要想通过实现 Callable 接口创建线程，就需要找到 Callable 接口与 Thread 类之间的关系。首先 FutureTask 类提供了构造方法 public FutureTask(`Callable<V> callable`)方法，而 FutureTask 类又实现了 RunnableFuture 接口，而 RunnableFuture 接口又继承了 Runnable 接口，再通过 public Thread(Runnable runnable)构造方法，使 Callable 接口与
Thread 类之间建立了联系。

所以使用 Callable 接口创建线程的步骤如下：

```java
// 1.定义一个类MyThread实现Callable接口，从而覆写call()方法
class MyThread implements Callable<String>{
    @Override
    public String call() throws Exception {
        return "Callable接口创建线程";
    }
}

public static void main(String[] args) throws InterruptedException,
ExecutionException {
        //2.利用MyThread类实例化Callable接口的对象
        Callable callable=new MyThread();
        //3.利用FutureTask类的构造方法public  FutureTask(Claaable<V>
callable)
        //将Callable接口的对象传给FutureTask类
        FutureTask task=new FutureTask(callable);
        //4.将FutureTask类的对象隐式地向上转型
        //从而作为Thread类的public Thread(Runnable runnable)构造方法的
参数
        Thread thread=new Thread(task);
        //5.调用Thread类的start()方法
        thread.start();
        //FutureTask的get()方法用于获取FutureTask的call()方法的返回值，
为了取得线程的执行结果
        System.out.println(task.get());
}
```

#### （2）就绪状态

在创建了线程之后，调用了 Thread 类的 start()方法来启动一个线程，即表示线程进入了就绪状态！

#### （3）运行状态

当线程获得 CPU 时间，线程才从就绪状态进入到运行状态！

#### （4）阻塞状态

线程进入运行状态后，可能由于多种原因让线程进入阻塞状态，如：调用 sleep()方法，让线程睡眠，调用 wait()方法让线程等待，调用 join()方法以及阻塞式 IO 方法

**阻塞情况分为三种：**
（一）等待阻塞：运行中（running）的线程执行 o.wait()方法，JVM 会把该线程放入等待队列（waiting queue）中。
（二）同步阻塞：运行中（running）的线程在获取对象的同步锁时，若该同步锁被别的线程占用，则 JVM 会把该线程放入锁池（lock pool）中。
（三）其他阻塞：运行中（running）的线程执行 Thread.sleep(long ms)或 t.join()方法，或者发出了 I/O 请求时，JVM 会把该线程置为阻塞状态。当 sleep()状态超时、join()等待线程终止或者超时、或者 I/O 处理完毕时，线程重新转入可运行状态。

**阻塞方法的异同：**

1. sleep、yield 方法是静态方法；作用是当前执行的线程；
2. yield 方法释放了 cpu 的执行权，但是保留了争夺 cpu 的资格，这也意味着，该线程可能马上会再次执行。yield()只能使同优先级或更高优先级的线程有执行的机会。
3. wait 释放 CPU 资源，同时释放锁，只有执行 notify/notifyAll()时，才会唤醒处于等待的线程，然后继续往下执行，直到执行完 synchronized 代码块的代码火种中途再次遇到 wait()。一般配合 synchronized 关键字使用，即，一般在 synchronized 同步代码块中使用 wait()、notify/notifyAll 方法。
4. sleep 释放 CPU 资源，但不释放锁
5. join 把调用该方法的线程加入到当前线程，可以将两个交替执行的线程合并为顺序执行的线程。比如在线程 B 中调用了线程 A 的 join()方法，直到线程 A 执行完毕后，才会继续执行线程 B

#### （5）死亡状态

run()方法的正常退出，就是让线程进入到死亡状态，还有当一个异常未被捕获而终止了 run()方法的执行也会进入到死亡状态！

### 设置或获取多线程的线程名称的方法

由于在一个进程中可能有多个线程，而多线程的运行状态又是不确定的，所以在多线程操作中需
要有一个明确的标识符标识出当前线程对象的信息，这个信息往往通过线程的名称来描述。在
Thread 类中提供了一些设置或者获取线程名称的方法：

1. 创建线程时设置线程的名称；
   public Thread(Runnable target,String name)
2. 设置线程名称的普通方法；
   public final synchronized void setName(String name)
3. 取得线程名称的普通方法；
   public final String getName()
