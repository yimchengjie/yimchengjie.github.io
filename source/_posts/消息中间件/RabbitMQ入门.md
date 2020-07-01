---
title: RabbitMQ入门
categories:
  - 消息中间件
tags:
  - 消息中间件
  - RabbitMQ
toc: true
date: 2019-08-25 21:55:17
---
## RabbitMQ入门

RabbitMQ是一个开源的机遇AMQP协议的实现,服务端用Erlang语言编写,支持多种客户端,用于在分布式系统中存储转发消息

### AMQP协议

### RabbitMQ的安装

1. 官网安装rabbitmq,注意需要安装erlang对应的版本
2. 安装完,可以直接启动服务`systemctl start rabbitmq-server`

### Web管理平台

1. 开启插件`rabbitmq-plugins enable rabbitmq_management`
2. 创建新用户,配置权限
    `rabbitmqctl add_user admin admin`
    分配操作权限
    `rabbitmqctl set_user_tags admin administrator`
    分配资源权限
    `rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"`

3. 开启防火墙

### java中的使用

```java
/**
 * 简单队列生产者
 * 使用RabbitMQ的默认交换器发送消息
 */
public class Producer {

    public static void main(String[] args) {
        // 1、创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        // 2、设置连接属性
        factory.setHost("192.168.100.242");
        factory.setPort(5672);
        factory.setUsername("admin");
        factory.setPassword("admin");

        Connection connection = null;
        Channel channel = null;

        try {
            // 3、从连接工厂获取连接
            connection = factory.newConnection("生产者");

            // 4、从链接中创建通道
            channel = connection.createChannel();

            /**
             * 5、声明（创建）队列
             * 如果队列不存在，才会创建
             * RabbitMQ 不允许声明两个队列名相同，属性不同的队列，否则会报错
             *
             * queueDeclare参数说明：
             * @param queue 队列名称
             * @param durable 队列是否持久化
             * @param exclusive 是否排他，即是否为私有的，如果为true,会对当前队列加锁，其它通道不能访问，并且在连接关闭时会自动删除，不受持久化和自动删除的属性控制
             * @param autoDelete 是否自动删除，当最后一个消费者断开连接之后是否自动删除
             * @param arguments 队列参数，设置队列的有效期、消息最大长度、队列中所有消息的生命周期等等
             */
            channel.queueDeclare("queue1", false, false, false, null);

            // 消息内容
            String message = "Hello World!";
            // 6、发送消息
            channel.basicPublish("", "queue1", null, message.getBytes());
            System.out.println("消息已发送！");

        } catch (IOException e) {
            e.printStackTrace();
        } catch (TimeoutException e) {
            e.printStackTrace();
        } finally {
            // 7、关闭通道
            if (channel != null && channel.isOpen()) {
                try {
                    channel.close();
                } catch (IOException e) {
                    e.printStackTrace();
                } catch (TimeoutException e) {
                    e.printStackTrace();
                }
            }

            // 8、关闭连接
            if (connection != null && connection.isOpen()) {
                try {
                    connection.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}


/**
 * 简单队列消费者
 */
public class Consumer {

    public static void main(String[] args) {
        // 1、创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        // 2、设置连接属性
        factory.setHost("192.168.100.242");
        factory.setUsername("admin");
        factory.setPassword("admin");

        Connection connection = null;
        Channel channel = null;

        try {
            // 3、从连接工厂获取连接
            connection = factory.newConnection("消费者");

            // 4、从链接中创建通道
            channel = connection.createChannel();

            /**
             * 5、声明（创建）队列
             * 如果队列不存在，才会创建
             * RabbitMQ 不允许声明两个队列名相同，属性不同的队列，否则会报错
             *
             * queueDeclare参数说明：
             * @param queue 队列名称
             * @param durable 队列是否持久化
             * @param exclusive 是否排他，即是否为私有的，如果为true,会对当前队列加锁，其它通道不能访问，
             *                  并且在连接关闭时会自动删除，不受持久化和自动删除的属性控制。
             *                  一般在队列和交换器绑定时使用
             * @param autoDelete 是否自动删除，当最后一个消费者断开连接之后是否自动删除
             * @param arguments 队列参数，设置队列的有效期、消息最大长度、队列中所有消息的生命周期等等
             */
            channel.queueDeclare("queue1", false, false, false, null);

            // 6、定义收到消息后的回调
            DeliverCallback callback = new DeliverCallback() {
                public void handle(String consumerTag, Delivery message) throws IOException {
                    System.out.println("收到消息：" + new String(message.getBody(), "UTF-8"));
                }
            };
            // 7、监听队列
            channel.basicConsume("queue1", true, callback, new CancelCallback() {
                public void handle(String consumerTag) throws IOException {
                }
            });

            System.out.println("开始接收消息");
            System.in.read();

        } catch (IOException e) {
            e.printStackTrace();
        } catch (TimeoutException e) {
            e.printStackTrace();
        } finally {
            // 8、关闭通道
            if (channel != null && channel.isOpen()) {
                try {
                    channel.close();
                } catch (IOException e) {
                    e.printStackTrace();
                } catch (TimeoutException e) {
                    e.printStackTrace();
                }
            }

            // 9、关闭连接
            if (connection != null && connection.isOpen()) {
                try {
                    connection.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
}
```
