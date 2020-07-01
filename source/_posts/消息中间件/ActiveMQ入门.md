---
title: ActiveMQ入门
categories:
  - 消息中间件
tags:
  - 消息中间件
  - ActiveMQ
toc: true
date: 2019-06-30 22:10:45
---
## ActiveMQ入门

ActiveMQ是Apache出品, 是一个完全支持JMS1.1和J2EE 1.4规范的JMS Provider实现
尽管JMS已经出台很久, 但在J2EE中仍然扮演者重要角色

### ActiveMQ特性

1. 支持多种编程语言
2. 支持多种传输协议
3. 支持多种持久化方式(文件系统/数据库)

### ActiveMQ的安装

1. 从官网下载ActiveMQ
2. 利用Xftp将文件传到服务器
3. 解压到`/var`目录下
4. 修改文件名为`activemq`

### ActiveMQ启动

在activemq文件下使用命令`./bin/activemq start` 后台启动ActiveMQ

但最好是将ActiveMQ作为一个服务来启动,这样可以使activemq在系统启动时自动启动

#### 创建ActiveMQ服务

1. 使用vim创建一个服务文件
   `vi /usr/lib/systemd/system/activemq.service`
2. 输入内容

    ```txt
    [Unit]
    Description=ActiveMQ service
    After=network.target

    [Service]
    Type=forking
    ExecStart=/var/activemq/bin/activemq start
    ExecStop=/var/activemq/bin/activemq stop
    User=root
    Group=root
    Restart=always
    RestartSec=9
    StandardOutput=syslog
    StandardError=syslog
    SyslogIdentifier=activemq

    [Install]
    WantedBy=multi-user.target
    ```

3. 修改`/var/activemq/bin/env`文件
    将JAVA_HOME的注释删除,然后填入系统jdk的安装路径

4. 启动ActiveMQ服务
    使用`systemctl start activemq`来启动服务

5. 查看服务状态
    使用命令`systemctl status activemq`

6. 设置开机自动启动
    `ln -s /usr/lib/systemd/system/activemq.service/etc/systemd/system/multi-user.target.wants/activemq.service`
    `systemctl enable activemq`

7. 在防火墙添加ActiveMQ的Web管理端口和通讯端口

    ```txt
    #Web管理端口默认为8161,通讯端口默认为61616
    ufw allow 8161
    ufw allow 61616
    # 部分服务器需要在对应的服务提供商管理页面修改出入站规则
    ```

8. 浏览器访问`http://服务IP:8161/admin`进入管理平台
   账户默认:admin
   密码默认:admin

#### Web管理平台配置

ActiveMQ的Web管理平台是基于jetty运行,因此在/var/activemq/conf目录可以查看jetty的配置文件

在`/var/activemq/conf/jetty.xml`文件中,可以进行修改端口,密码等内容

### 使用ActiveMQ

在Java中使用

```xml
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-all</artifactId>
    <version>5.15.8</version>
</dependency>
```

```java
/**
 * 简单生产者
 */
public class Producer {
    public static void main(String[] args) {
        new ProducerThread("tcp://119.3.218.159:61616", "queue1").start();
    }

    static class ProducerThread extends Thread {
        String brokerUrl;
        String destinationUrl;

        // brokerUrl中间件url地址, destinationUrl队列url
        public ProducerThread(String brokerUrl, String destinationUrl) {
            this.brokerUrl = brokerUrl;
            this.destinationUrl = destinationUrl;
        }

        @Override
        public void run() {
            ActiveMQConnectionFactory connectionFactory;
            Connection conn;
            Session session;

            try {
                // 1、创建连接工厂
                connectionFactory = new ActiveMQConnectionFactory(brokerUrl);
                // 2、创建连接对象md
                conn = connectionFactory.createConnection();
                conn.start();
                // 3、创建会话
                session = conn.createSession(false, Session.AUTO_ACKNOWLEDGE);
                // 4、创建点对点发送的目标
                Destination destination = session.createQueue(destinationUrl);
                // 5、创建生产者消息
                MessageProducer producer = session.createProducer(destination);
                // 设置生产者的模式，有两种可选 持久化 / 不持久化
                producer.setDeliveryMode(DeliveryMode.PERSISTENT);
                // 6、创建一条文本消息
                String text = "Hello world!";
                TextMessage message = session.createTextMessage(text);
                for (int i = 0; i < 1; i++) {
                    // 7、发送消息
                    producer.send(message);
                }
                // 8、 关闭连接
                session.close();
                conn.close();
            } catch (JMSException e) {
                e.printStackTrace();
            }
        }
    }
}


/**
 * 简单消费者
 */
// http://activemq.apache.org/consumer-features.html
public class Consumer {
    public static void main(String[] args) {
        new ConsumerThread("tcp://119.3.218.159:61616", "queue1").start();
        new ConsumerThread("tcp://119.3.218.159:61616", "queue1").start();
    }

    static class ConsumerThread extends Thread {

        String brokerUrl;
        String destinationUrl;

        public ConsumerThread(String brokerUrl, String destinationUrl) {
            this.brokerUrl = brokerUrl;
            this.destinationUrl = destinationUrl;
        }

        @Override
        public void run() {
            ActiveMQConnectionFactory connectionFactory;
            Connection conn;
            Session session;
            MessageConsumer consumer;

            try {
                // brokerURL http://activemq.apache.org/connection-configuration-uri.html
                // 1、创建连接工厂
                connectionFactory = new ActiveMQConnectionFactory(this.brokerUrl);
                // 2、创建连接对象
                conn = connectionFactory.createConnection();
                conn.start(); // 一定要启动
                // 3、创建会话（可以创建一个或者多个session）
                session = conn.createSession(false, Session.AUTO_ACKNOWLEDGE);
                // 4、创建点对点接收的目标，queue - 点对点
                Destination destination = session.createQueue(destinationUrl);

                // 5、创建消费者消息 http://activemq.apache.org/destination-options.html
                consumer = session.createConsumer(destination);

                // 6、接收消息(没有消息就持续等待)
                Message message = consumer.receive();
                if (message instanceof TextMessage) {
                    System.out.println("收到文本消息：" + ((TextMessage) message).getText());
                } else {
                    System.out.println(message);
                }

                consumer.close();
                session.close();
                conn.close();
            } catch (JMSException e) {
                e.printStackTrace();
            }
        }
    }
}
```

在Spring中使用

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jms</artifactId>
    <version>5.1.3.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.apache.activemq</groupId>
    <artifactId>activemq-broker</artifactId>
    <version>5.15.8</version>
    <exclusions>
    <exclusion>
    <groupId>geronimo-jms_1.1_spec</groupId>
    <artifactId>org.apache.geronimo.specs</artifactId>
    </exclusion>
    </exclusions>
</dependency>
```

### web控制台

activemq支持web控制台
`http://host:8161/admin` 默认账号admin,密码admin

### 持久化

ActiveMQ的消息持久化机制有JDBC，AMQ,KahaDB和LevelDB，无论使用哪种持久化，消息的存储逻辑都是一致的

1. Queue(消息队列)类型的持久化机制
2. Topic(消息订阅)类型的持久化机制

#### 持久化机制

1. JDBC: 存入数据库,方便管理,性能低
2. AMQ: 基于文件的存储方式,写入速度快,且易于恢复,但是建索引时间长
3. KahaDB: 默认方式,相比AMQ恢复更快,并且占用数据量更少
4. LevelDB: 谷歌开发的持久化高性能类库.
