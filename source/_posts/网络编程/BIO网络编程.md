---
title: BIO网络编程
categories:
  - 网络编程
tags:
  - 网络编程
toc: true
date: 2019-04-30 19:29:55
---

## BIO 网络编程

同步阻塞网络编程

### 阻塞 IO 的含义

阻塞 IO: 资源不可用时,IO 请求一直阻塞,直到有数据或者阻塞超时
非阻塞 IO: 资源不可用时,IO 请求离开返回,返回数据标识资源不可用
阻塞非阻塞描述请求,即如何获取数据

同步 IO: 应用阻塞在发送或者接受数据的状态,直到数据成功传输或者返回失败
异步 IO: 应用发送或接受数据后立刻返回,实际处理是异步执行
同步异步描述响应,即如何拿数据

Java 中的
ServerSocket 类的 accept 方法和 InputStream 类的 read 方法,都是阻塞 API

```java
// 服务端
// 初始化服务端Socket
ServerSocket serverSocket = new ServerSocket(8080);
System.out.println("服务器启动成功");
while (!serverSocket.isClosed()) {
    Socket request = serverSocket.accept();// 阻塞
    System.out.println("收到新连接 : " + request.toString());
    try {
        // 接收数据、打印
        InputStream inputStream = request.getInputStream();
        BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream, "utf-8"));
        String msg;
        while ((msg = reader.readLine()) != null) { // 没有数据，阻塞
            if (msg.length() == 0) {
                break;
            }
            System.out.println(msg);
        }
        System.out.println("收到数据,来自："+ request.toString());
    } catch (IOException e) {
        e.printStackTrace();
    } finally {
        try {
            request.close();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
serverSocket.close();

// 客户端
Socket s = new Socket("localhost", 8080);
OutputStream out = s.getOutputStream();

Scanner scanner = new Scanner(System.in);
System.out.println("请输入：");
String msg = scanner.nextLine();
out.write(msg.getBytes(charset)); // 阻塞，写完成
scanner.close();
s.close();
```
