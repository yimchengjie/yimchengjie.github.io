---
title: NIO
categories:
  - 网络编程
tags:
  - 网络编程
toc: true
date: 2019-04-30 22:19:55
---
## NIO

非阻塞 IO

NIO 是 Java 提供的替代 BIO 的相关 API

### NIO 三大核心组件

Buffr 缓冲区
Channel 通道
Selector 选择器

#### Buffer 缓冲区

Java 提供 Buffer API, 可以让我们更轻松的使用内存块

使用 Buffer 对象,对数据进行写入和读取

1. 将数据写入缓冲区
2. 调用 buffer.flip(),转换为读取模式
3. 缓冲区读取数据
4. 调用 buffer.clear()或 buffer.compact() 清楚缓冲区

Buffer 的三个属性
capacity 容量: 缓冲区内存块大小
position 位置: 写入或读取时的位置
limit 限制: 限制每次读取或者写入的大小

```java
public class BufferDemo {
    public static void main(String[] args) {
        // 构建一个byte字节缓冲区，容量是4
        ByteBuffer byteBuffer = ByteBuffer.allocateDirect(4);
        // 默认写入模式，查看三个重要的指标
        System.out.println(String.format("初始化：capacity容量：%s, position位置：%s, limit限制：%s", byteBuffer.capacity(),
                byteBuffer.position(), byteBuffer.limit()));
        // 写入2字节的数据
        byteBuffer.put((byte) 1);
        byteBuffer.put((byte) 2);
        byteBuffer.put((byte) 3);
        // 再看数据
        System.out.println(String.format("写入3字节后，capacity容量：%s, position位置：%s, limit限制：%s", byteBuffer.capacity(),
                byteBuffer.position(), byteBuffer.limit()));

        // 转换为读取模式(不调用flip方法，也是可以读取数据的，但是position记录读取的位置不对)
        System.out.println("#######开始读取");
        byteBuffer.flip();
        byte a = byteBuffer.get();
        System.out.println(a);
        byte b = byteBuffer.get();
        System.out.println(b);
        System.out.println(String.format("读取2字节数据后，capacity容量：%s, position位置：%s, limit限制：%s", byteBuffer.capacity(),
                byteBuffer.position(), byteBuffer.limit()));

        // 继续写入3字节，此时读模式下，limit=3，position=2.继续写入只能覆盖写入一条数据
        // clear()方法清除整个缓冲区。compact()方法仅清除已阅读的数据。转为写入模式
        byteBuffer.compact(); // buffer : 1 , 3
        byteBuffer.put((byte) 3);
        byteBuffer.put((byte) 4);
        byteBuffer.put((byte) 5);
        System.out.println(String.format("最终的情况，capacity容量：%s, position位置：%s, limit限制：%s", byteBuffer.capacity(),
                byteBuffer.position(), byteBuffer.limit()));

        // rewind() 重置position为0
        // mark() 标记position的位置
        // reset() 重置position为上次mark()标记的位置

    }
}
```

Buffer 可以直接获取直接内存
ByteBuffer directByteBuffer=ByteBuffer.allocateDirect(n);
内部有一个回收对象, 可以进行垃圾回收, 否则 JVM 的垃圾回收无法管理堆外内存

使用直接内存， 可以减少一次数据拷贝， 如果使用 JVM 堆内存,写入时用堆内存会复制一份数据到堆外内存。 因为JVM进行GC时会移动数据的位置， 导致IO写入异常

#### Channel 通道

通道从 ByteBuffer 中读取数据或者写入数据

##### Channel 四种实现类型

**1. FileChannel**: 用于文件的数据读写。

```java
// 创建FileChannel通道
RandomAccessFile aFile = new RandomAccessFile("test.txt"，"rw");
FileChannel inChannel = aFile.getChannel();
// 读取数据
ByteBuffer buf = ByteBuffer.allocate(48);
int bytesRead = inChannel.read(buf);
// 写入数据
String newData = "New String to write to file..." + System.currentTimeMillis();
ByteBuffer buf = ByteBuffer.allocate(48);
buf.clear();
buf.put(newData.getBytes());
buf.flip();
while(buf.hasRemaining()){
    channel.write(buf);
}
// 关闭
channel.close();
```

**2. DatagramChannel**: 用于 UDP 的数据读写。
**3. SocketChannel**: 用于 TCP 的数据读写。
**4. ServerSocketChannel**: 监听 TCP 链接请求，每个请求会创建会一个 SocketChannel。

```java
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
serverSocketChannel.socket().bind(new InetSocketAddress(9999));
serverSocketChannel.configureBlocking(false);
while(true){
    SocketChannel socketChannel = serverSocketChannel.accept();
    if(socketChannel != null){
        //do something with socketChannel...
    }
}
```

#### Selector 选择器

可以检查一个或多个 NIO 通道,实现单个线程管理多个通道,从而管理多个网络连接

比如:当线程从某客户端 Socket 通道进行读写数据时，若没有数据可用时，该线程可以进行其他任务,不会阻塞

一个通道继承了 SelectableChannel,那么他就可以被 Selector 复用

**一个通道可以被注册到多个选择器上，但对每个选择器而言只能被注册一次。**
通道和选择器之间的关系，使用注册的方式完成。SelectableChannel 可以被注册到 Selector 对象上，在注册的时候，需要指定通道的哪些操作，是 Selector 感兴趣的。

使用 Channel.register（Selector sel，int ops）方法, 将通道注册到选择器上,这里的操作指的是当前通道已经准备就绪,能够进行的操作类型
int ops 包括

1. 可读 : SelectionKey.OP_READ
2. 可写 : SelectionKey.OP_WRITE
3. 连接 : SelectionKey.OP_CONNECT
4. 接收 : SelectionKey.OP_ACCEPT

`selector.select();` 查找准备就绪的通道

##### Selector 使用流程

```java
//创建选择器
Selector selector = Selector.open();

//创建通道
ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
//设置非租塞
serverSocketChannel.configureBlocking(false);
//绑定连接
serverSocketChannel.bind(new InetSocketAddress(SystemConfig.SOCKET_SERVER_PORT));

//将通道注册到选择器,并指定为可接收
serverSocketChannel.register(selector，SelectionKey.OP_ACCEPT);

// 采用轮询的方式，查询获取“准备就绪”的注册过的操作
while (selector.select() > 0)
{
    // 获取当前选择器中所有注册的选择键（“已经准备就绪的操作”）
    Iterator<SelectionKey> selectedKeys = selector.selectedKeys().iterator();
    while (selectedKeys.hasNext())
    {
        // 获取“准备就绪”的事件
        SelectionKey selectedKey = selectedKeys.next();
        // 判断key是具体的什么事件
        if (selectedKey.isAcceptable())
        {
            // 若接受的事件是“接收就绪” 操作,就获取客户端连接
            SocketChannel socketChannel = serverSocketChannel.accept();
            // 切换为非阻塞模式
            socketChannel.configureBlocking(false);
            // 将该通道注册到selector选择器上,并指定为可读
            socketChannel.register(selector, SelectionKey.OP_READ);
        }
        else if (selectedKey.isReadable())
        {
            // 获取该选择器上的“读就绪”状态的通道
            SocketChannel socketChannel = (SocketChannel) selectedKey.channel();
            // 读取数据
            ByteBuffer byteBuffer = ByteBuffer.allocate(1024);
            int length = 0;
            while ((length = socketChannel.read(byteBuffer)) != -1){
                byteBuffer.flip();
                System.out.println(new String(byteBuffer.array(), 0, length));
                byteBuffer.clear();
            }
            socketChannel.close();
        }
        // 移除选择键
        selectedKeys.remove();
    }
}
// 关闭连接
serverSocketChannel.close();
}
```

**注意**:要注册到选择器, 通道必须是非租塞的

### Reactor 模式

基于 Java NIO, 在此基础, 抽象出来两个组件--Reactor 和 Handler

1. Reactor: 负责响应 IO 事件,当检测到新的时间, 发送给相应的 Handler
2. Handler: 执行处理
 