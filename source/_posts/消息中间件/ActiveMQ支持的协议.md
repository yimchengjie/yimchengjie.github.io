---
title: ActiveMQ支持的协议
categories:
  - 消息中间件
tags:
  - 消息中间件
  - ActiveMQ
toc: true
date: 2019-07-30 19:40:31
---
## ActiveMQ支持的协议

ActiveMQ支持多种协议传输和传输方式,允许客户端使用多种协议连接.

ActiveMQ支持的协议: AUTO、OpenWire、AMQP、Stomp、MQTT等
在`${ACTIVE_HOME}/conf/activemq.xml`中, 通过配置`<transportConnectors>`就可以使用多种传输方式(==注意配置文件中可能会用到转义字符串, 比如&要用&amp;来表示==)

ActiveMQ支持的基础传输方式有: VM、TCP、SSL、UDP、Peer、Multicast、HTTP(S)等

由于TCP的稳定性, 它也是ActiveMQ中最常用的一种传输方式. 在默认的设置中, TCP连接的端口为61616

### OpenWire协议

OpenWire协议是Apache的一种跨语言的协议,允许从不同的语言和平台访问ActiveMQ
默认就是使用这种协议, 功能很全面

OpenWire支持TCP、SSL、NIO、UDP、VM等传输方式，但URL只配置传输方式时，默认就是使用OpenWire协议

### MQTT协议

全称Message Queuing Telemetry Transport，即消息队列遥测传输，由IBM开发， 已成为物联网通信的标准

它结构简单，相比其他的协议更加轻量级

#### 发布-订阅模型

MQTT协议使用发布-订阅模型
提供者发布消息到主题topic上, 只要订阅了这个topic的消费者,都能收到这条消息

消费者无法收到启动前topic上的消息

#### MQTT中的服务质量(QoS)

服务质量(QoS)级别 是一种关于发送者和接受者之间信息传递的保证协议

MQTT支持三种QoS
至多一次(0)客户端只发布一次消息到服务器
至少一次(1)客户端发送消息,直到服务器返回成功
只有一次(2)在(1)的前提下, 客户端继续发送, 客户端查看是否存在, 有就删除

QoS是的在不可靠的网络下进行通信变得更加简单,因为即使是在非常不可靠的网络下,协议也可以掌控是否需要重发消息, 并保证消息到达

#### ActiveMQ中使用MQTT协议

```java
//发布者
public class MqttProducer {
    private static int qos = 1;
    private static String broker = "tcp://activemq.tony.com:1883";
    private static String userName = "admin";
    private static String passWord = "admin";

    private static MqttClient connect(String clientId, String userName,String password) throws MqttException {
        MemoryPersistence persistence = new MemoryPersistence();
        MqttConnectOptions connOpts = new MqttConnectOptions();
        connOpts.setCleanSession(true);
        connOpts.setUserName(userName);
        connOpts.setPassword(password.toCharArray());
        connOpts.setConnectionTimeout(10);
        connOpts.setKeepAliveInterval(20);
        // connOpts.setServerURIs(uris);  //这个是mqtt客户端实现的负载均衡和容错
        // String[] uris = {"tcp://10.100.124.206:1883","tcp://10.100.124.207:1883"};
        MqttClient mqttClient = new MqttClient(broker, clientId, persistence);
        mqttClient.setCallback(new PushCallback("test"));
        mqttClient.connect(connOpts);
        return mqttClient;
    }


    private static void pub(MqttClient sampleClient, String msg, String topic)
            throws Exception {
        MqttMessage message = new MqttMessage(msg.getBytes());
        message.setQos(qos);
        message.setRetained(false);
        sampleClient.publish(topic, message);
    }

    private static void publish(String str, String clientId, String topic) throws Exception {
        MqttClient mqttClient = connect(clientId, userName, passWord);
        if (mqttClient != null) {
            pub(mqttClient, str, topic);
            System.out.println("pub-->" + str);
        }
        if (mqttClient != null) {
            mqttClient.disconnect();
        }
    }

    public static void main(String[] args) throws Exception {
        publish("message content", "producer-client-id-0", "x/y/z");
    }
}

class PushCallback implements MqttCallback {
    private String threadId;

    public PushCallback(String threadId) {
        this.threadId = threadId;
    }

    public void connectionLost(Throwable cause) {
        cause.printStackTrace();
    }

    public void deliveryComplete(IMqttDeliveryToken token) {
        System.out.println("服务器是否正确接收---------" + token.isComplete());
    }

    public void messageArrived(String topic, MqttMessage message) throws Exception {
        String msg = new String(message.getPayload());
        System.out.println(threadId + " " + msg);
    }
}


//消费者
public class MqttConsumer {
    private static int qos = 2;
    private static String broker = "tcp://activemq.tony.com:1883";
    private static String userName = "admin";
    private static String passWord = "admin";

    private static MqttClient connect(String clientId) throws MqttException {
        MemoryPersistence persistence = new MemoryPersistence();
        MqttConnectOptions connOpts = new MqttConnectOptions();
        connOpts.setCleanSession(false);
        connOpts.setUserName(userName);
        connOpts.setPassword(passWord.toCharArray());
        connOpts.setConnectionTimeout(10);
        connOpts.setKeepAliveInterval(20);
        MqttClient mqttClient = new MqttClient(broker, clientId, persistence);
        mqttClient.connect(connOpts);
        return mqttClient;

    }

    public static void sub(MqttClient mqttClient, String topic) throws MqttException {
        int[] Qos = {qos};
        String[] topics = {topic};
        mqttClient.subscribe(topics, Qos, new IMqttMessageListener[]{(s, mqttMessage) -> {
            System.out.println("收到新消息" + s + " > " + mqttMessage.toString());
        }});
    }

    private static void runsub(String clientId, String topic) throws MqttException {
        MqttClient mqttClient = connect(clientId);
        if (mqttClient != null) {
            sub(mqttClient, topic);
        }
    }

    public static void main(String[] args) throws MqttException {
        runsub("consumer-client-id-1", "x/y/z");
    }

}

```

### AUTO协议

AUTO自动检测协议,可以自动检测ActiveMQ支持的所有协议, 允许使用各种协议的客户端,使用同一个传输

### Stomp协议

可以使用webSocket传输协议
