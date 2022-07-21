---
title: RabbitMQ架构和消息分发机制
date: 2022-07-09 08:30:37
categories:
- 中间键
- 消息中间键
tag:
top:
keywords:
description:
cover: 'https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207101040133.png'
top_img: 'https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207101040133.png'
---

# 简介

Erlang 是为电话 交换机编写的语言，天生适合分布式和高并发。

国内的绝大部分大厂都在用 RabbitMQ，包括头条，美团，滴滴（TMD），去哪儿， 艺龙，淘宝也有用。 

RabbitMQ 和 Spring 家族属于同一家公司：Pivotal。 

除了 AMQP 之外，RabbitMQ 支持多种协议，STOMP、MQTT、HTTP、 WebSockets

# 部署

使用Docker部署，参考博客Centos项目搭建（本地搜索即可）



# RabbitMQ架构原理

预览；连接；channels-通道；Exchanges-交换机；队列；用户。

RabbitMQ 实现了 AMQP 协议，所以 RabbitMQ 的工作模型也是基于 AMQP 的。

![image-20220711093357648](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207110933720.png)

## **Broker 主机**

代理/中介主机（RabbitMQ服务也就是Broker，作用就是存储、转发消息）

## **Connection 连接** 

无论是生产者发送消息，还是消费者接收消息，都必须要跟 Broker 之间建立一个 连接，这个连接是一个 TCP 的长连接。

## **Channel 通道**

AMQP 里面引入了 Channel 的概念，它是一个虚拟的连接。我们把它翻 译成通道，或者消息信道。这样我们就可以在保持的 TCP 长连接里面去创建和释放 Channel，大大了减少了资源消耗。

Channel 是 RabbitMQ 原生 API 里面的最重要的编程接 口，也就是说我们定义交换机、队列、绑定关系，发送消息，消费消息，调用的都是 Channel 接口上的方法。

## **Queue 队列**

连接到 Broker 以后，就可以收发消息了。 在 Broker 上有一个对象用来存储消息，在 RabbitMQ 里面这个对象叫做 Queue。 

实际上 RabbitMQ 是用数据库来存储消息的。

CentOS 保存在/var/lib 目录下： /var/lib/rabbitmq/mnesia

**队列也是生产者和消费者的纽带，生产者发送的消息到达队列，在队列中存储。 消费者从队列消费消息。**

![image-20220713141058687](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207131410895.png)

![image-20220713141120505](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207131411568.png)

## Consumer 消费者

消息到底是 Broker 推送给消费者的？还是消费者主动获取的？

消费者消费消息 有两种模式。 

一种是 Pull 模式，对应的方法是 basicGet。消息存放在服务端，只有消费者**主动 获取才能拿到消息**。如果每隔一段时间获取一次消息，消息的**实时性会降低**。但是好 处是可以根据自己的消费能力决定获取消息的频率。 

另一种是 Push 模式，对应的方法是 basicConsume，只要生产者发消息到服务 器，就马上推送给消费者，消息保存在客户端，实时性很高，如果消费不过来有可能 会造成**消息积压**。Spring AMQP 是 push 方式，通过事件机制**对队列进行监听**，只要 有消息到达队列，就会触发消费消息的方法。

**而 kafka 和 RocketMQ 只有 pull**。

由于队列有 FIFO（先进先出） 的特性，只有确定前一条消息被消费者接收之后，Broker 才会 把这条消息从数据库删除，继续投递下一条消息。

一个消费者是可以监听多个队列的，一个队列也可以被多个消费者监听。

## Exchange 交换机

为什莫要有交换机？

如果要把一条消息发送给多个队列，被多个消费者消 费，应该怎么做？生产者是不是必须要调用多次 basicPublish 的方法，依次发送给多 个队列？就像消息推送的这种场景，有成千上万个队列的时候，对生产者来说压力太 大了。

所以交换机就是消息路由的组件。交换机的作用就是把接收到的消息分发给队列。

所以队列和交换机需要绑定。关系可以是多对多的关系。

**绑定关系建立好之后，生产者发送消息到 Exchange，也会携带一个特殊的标识。 当这个标识跟绑定的标识匹配的时候，消息就会发给一个或者多个符合规则的队列**

![image-20220713190103949](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207131901025.png)

![image-20220713190138034](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207131901092.png)



## Vhost 虚拟主机

就是Admin的角色，可以提供给不同的用户，用户可以访问自己的交换机和队列，超级管理可以访问所有的vhost权限。

目的就是和其他角色隔离，可以拥有自己的broke。

# 消息分发机制

就是Exchange的主要功能，RabbitMQ 中一共有四种类型的交换机，Direct、Topic、Fanout、Headers。

## Direct 直连

一个队列与直连类型的交换机绑定，需指定一个明确的绑定键（binding key）。

生产者发送消息时会携带一个路由键（routing key）。

当消息的路由键与某个队列的绑定键完全匹配时，这条消息才会从交换机路由到 这个队列上。多个队列也可以使用相同的绑定键。

![image-20220714174412655](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207141744823.png)

```java
channel.basicPublish(“MY_DIRECT_EXCHANGE”,”spring”,”msg 1”);

//只有spring队列可以接收消息（明确指定给一个队列）
```



## Topic 主题

**适用业务主题或者消息等级需要过滤消息的场景**

一个队列与主题类型的交换机绑定时，可以在绑定键中使用通配符。支持两个通 配符：

```text
# 代表匹配 0 个或者多个单词 
* 代表匹配不多不少一个单词
单词（word）指的是用英文的点“.”隔开的字符。例如 a.bc.def 是 3 个单词

第一个队列支持路由键以 junior 开头的消息路由，后面可以有单词，也可以没有。
第二个队列支持路由键以 netty 结尾，前面可以有也可以没有单词的消息路由。
第三个队列支持路由键以 senior 开头，并且后面是一个单词的消息路由。
第四个队列支持路由键以 jvm结尾，并且前面是一个单词的消息路由。
```

![image-20220714174800079](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207141748152.png)

```java
channel.basicPublish("MY_TOPIC_EXCHANGE","junior.abc.jvm","msg 2"); 
//只有第一个队列能收到消息。 
channel.basicPublish("MY_TOPIC_EXCHANGE","senior.netty", "msg 3"); 
//第二个队列和第三个队列能收到消息。
```



## Fanout 广播

**适用于一些通用的业务消息**

广播类型的交换机与队列绑定时，不需要指定绑定键。

```java
channel.basicPublish("MY_FANOUT_EXCHANGE", "", "msg 4")
```

# RabbitMQ 持久化与内存管理

## 持久化机制

RabbitMQ 的持久化分为消息持久化、队列持久化、交换器持久化。无论是持久 化消息还是非持久化消息都可以被写入磁盘。

当 RabbitMQ 收到消息时，如果是持久化消息，则会储存在内存中，同时也会写 入磁盘；如果是非持久化消息，则只会存在内存中。当内存使用达到 RabbitMQ 的临 界值时，内存中的数据会被交换到磁盘，持久化消息由于本就存在于磁盘中，不会被 重复写入。**（也就是上面无论是否持久化都可以被写入磁盘，条件是是否达到内存使用临界值）**

但是非持久化的消息、队列、交换机，重启会被清除。

## 内存控制 

RabbitMQ 中通过内存阈值参数控制内存的使用量，当内存使用超过配置的阈值 时，RabbitMQ 会阻塞客户端的连接并停止接收从客户端发来的消息，以免服务崩溃， 同时，会发出内存告警，此时客户端于与服务端的心跳检测也会失效。

可以通过，rabbitmq.conf。

```
# rabbitmq.conf
vm_memory_high_watermark.relative=0.4
#vm_memory_high_watermark.absolute=1GB

/*
RabbitMQ 提供 relative 与 absolute 两种配置方式 relative：相对值，也就是前面的 fraction 参数，建议 0.4～0.66，不能太大 absolute：绝对值，固定大小，单位为 KB、MB、GB
*/

rabbitmqctl set_vm_memory_high_watermark absolute/0.4
```

## 内存换页 

在 RabbitMQ 达到内存阈值并阻塞生产者之前，会尝试**<u>将内存中的消息换页到磁 盘，以释放内存空间</u>**。内存换页由换页参数控制，默认为 0.5，表示当内存使用量达到 内存阈值的 50%时会进行换页，也就是 0.4*0.5=0.2。 

```
vm_memory_high_watermark_paging_ratio=0.5 

#当换页阈值大于 1 时，相当于禁用了换页功能
```

## 磁盘控制

RabbitMQ 通过磁盘阈值参数控制磁盘的使用量，当磁盘剩余空间小于磁盘阈值 时，RabbitMQ 同样会阻塞生产者，避免磁盘空间耗尽。 

磁盘阈值默认 50M，由于是定时检测磁盘空间，不能完全消除因磁盘耗尽而导致 崩溃的可能性，比如在两次检测之间，磁盘空间从大于 50M 变为 0M。

 一种相对谨慎的做法是将磁盘阈值大小设置与内存相等

```
rabbitmqctl set_disk_free_limit <limit>
rabbitmqctl set_disk_free_limit mem_relative <fraction>
# limit 为绝对值，KB、MB、GB
# fraction 为相对值，建议 1.0～2.0 之间

# rabbitmq.conf
disk_free_limit.relative=1.5
# disk_free_limit.absolute=50MB
```



**配置永久生效只有去更改配置文件rabbitmq.conf**

# RabbiMQ 插件管理

https://www.rabbitmq.com/configure.html

RabbitMQ 插件机制的设计，主要是用于面对特殊场景的需求，可以实现任意扩展。

```
//插件前面[ ] 为空说明，没有安装。有 e*说明插件是安装了的
rabbitmq-plugins list
//是安装启用管理
rabbitmq-plugins enable rabbitmq_management
//插件卸载
rabbitmq-plugins disable rabbitmq_management
//查看 rabbitmq 的有效配置
rabbitmqctl environment
```

# API介绍

## JAVA基础API

![image-20220721173548923](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207211735027.png)

```api
<dependency>
    <groupId>com.rabbitmq</groupId>
    <artifactId>amqp-client</artifactId>
    <version>5.6.0</version>
</dependency>
```

```java
package rabbitmq.javaapi.simple;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

/**
 * 消息生产者
 */
public class MyProducer {
    private final static String EXCHANGE_NAME = "GP_SIMPLE_EXCHANGE_FORM_JAVAAPI";

    public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        // 连接IP
        factory.setHost("192.168.8.147");
        // 连接端口
        factory.setPort(5672);
        // 虚拟机
        factory.setVirtualHost("/");
        // 用户
        factory.setUsername("admin");
        factory.setPassword("123456");

        // 建立连接
        Connection conn = factory.newConnection();
        // 创建消息通道
        Channel channel = conn.createChannel();

        // 发送消息
        String msg = "Hello world, Rabbit MQ";

        // String exchange, String routingKey, BasicProperties props, byte[] body
        channel.basicPublish(EXCHANGE_NAME, "gupao.best", null, msg.getBytes());

        channel.close();
        conn.close();
    }
}


```

```java
package rabbitmq.javaapi.simple;

import com.rabbitmq.client.*;

import java.io.IOException;

/**
 * 消息消费者
 */
public class MyConsumer {
    private final static String EXCHANGE_NAME = "GP_SIMPLE_EXCHANGE_FORM_JAVAAPI";
    private final static String QUEUE_NAME = "GP_SIMPLE_QUEUE_FORM_JAVAAPI";

    public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        // 连接IP
        factory.setHost("192.168.8.147");
        // 默认监听端口
        factory.setPort(5672);
        // 虚拟机
        factory.setVirtualHost("/");

        // 设置访问的用户
        factory.setUsername("admin");
        factory.setPassword("123456");
        // 建立连接
        Connection conn = factory.newConnection();
        // 创建消息通道
        Channel channel = conn.createChannel();

        // 声明交换机
        // String exchange, String type, boolean durable, boolean autoDelete, Map<String, Object> arguments
        channel.exchangeDeclare(EXCHANGE_NAME,"direct",false, false, null);

        // 声明队列
        // String queue, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object> arguments
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        System.out.println(" Waiting for message....");

        // 绑定队列和交换机
        channel.queueBind(QUEUE_NAME,EXCHANGE_NAME,"gupao.best");

        // 创建消费者
        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties,
                                       byte[] body) throws IOException {
                String msg = new String(body, "UTF-8");
                System.out.println("Received message : '" + msg + "'");
                //消费者标记，第一次请求获得，后续保持不变
                System.out.println("consumerTag : " + consumerTag );
                //投递消息的序号
                System.out.println("deliveryTag : " + envelope.getDeliveryTag() );
            }
        };

        // 开始获取消息
        // String queue, boolean autoAck, Consumer callback
        channel.basicConsume(QUEUE_NAME, true, consumer);
    }
}


```

### 理解

>消费者：先通过ConnectFactory，set基本设置IP、端口、用户、密码，然后建立连接，创建通道，通道声明交换机、队列，绑定队列和交换机，创建消费者，获取消息。
>
>生产者：先通过ConnectFactory，set基本设置IP、端口、用户、密码，然后建立连接，创建通道，发送消息（指定队列名，交换机名消息体）

## Spring AMQP API

![image-20220721212538174](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207212125255.png)

```
<!--rabbitmq依赖 -->
<dependency>
    <groupId>org.springframework.amqp</groupId>
    <artifactId>spring-rabbit</artifactId>
    <version>1.3.5.RELEASE</version>
</dependency>
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:rabbit="http://www.springframework.org/schema/rabbit"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
     http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
     http://www.springframework.org/schema/rabbit
     http://www.springframework.org/schema/rabbit/spring-rabbit-1.2.xsd">

    <!--配置connection-factory，指定连接rabbit server参数 -->
    <rabbit:connection-factory id="connectionFactory" virtual-host="/" username="admin" password="123456" host="192.168.8.147" port="5672" />

    <!--通过指定下面的admin信息，当前producer中的exchange和queue会在rabbitmq服务器上自动生成 -->
    <rabbit:admin id="connectAdmin" connection-factory="connectionFactory" />

    <!--######分隔线######-->
    <!--定义queue -->
    <rabbit:queue name="GP_FIRST_QUEUE_FORM_SPRING" durable="true" auto-delete="false" exclusive="false" declared-by="connectAdmin" />

    <!--定义direct exchange，绑定GP_FIRST_QUEUE -->
    <rabbit:direct-exchange name="GP_DIRECT_EXCHANGE_FORM_SPRING" durable="true" auto-delete="false" declared-by="connectAdmin">
        <rabbit:bindings>
            <rabbit:binding queue="GP_FIRST_QUEUE_FORM_SPRING" key="FirstKey">
            </rabbit:binding>
        </rabbit:bindings>
    </rabbit:direct-exchange>

    <!--定义rabbit template用于数据的接收和发送 -->
    <rabbit:template id="amqpTemplate" connection-factory="connectionFactory" exchange="GP_DIRECT_EXCHANGE_FORM_SPRING" />

    <!--消息接收者 -->
    <bean id="messageReceiver" class="rabbitmq.springapi.consumer.FirstConsumer"></bean>

    <!--queue listener 观察 监听模式 当有消息到达时会通知监听在对应的队列上的监听对象 -->
    <rabbit:listener-container connection-factory="connectionFactory">
        <rabbit:listener queues="GP_FIRST_QUEUE_FORM_SPRING" ref="messageReceiver" />
    </rabbit:listener-container>

    <!--定义queue -->
    <rabbit:queue name="GP_SECOND_QUEUE_FORM_SPRING" durable="true" auto-delete="false" exclusive="false" declared-by="connectAdmin" />

    <!-- 将已经定义的Exchange绑定到GP_SECOND_QUEUE，注意关键词是key -->
    <rabbit:direct-exchange name="GP_DIRECT_EXCHANGE_FORM_SPRING" durable="true" auto-delete="false" declared-by="connectAdmin">
        <rabbit:bindings>
            <rabbit:binding queue="GP_SECOND_QUEUE_FORM_SPRING" key="SecondKey"></rabbit:binding>
        </rabbit:bindings>
    </rabbit:direct-exchange>

    <!-- 消息接收者 -->
    <bean id="receiverSecond" class="rabbitmq.springapi.consumer.SecondConsumer"></bean>

    <!-- queue litener 观察 监听模式 当有消息到达时会通知监听在对应的队列上的监听对象 -->
    <rabbit:listener-container connection-factory="connectionFactory">
        <rabbit:listener queues="GP_SECOND_QUEUE_FORM_SPRING" ref="receiverSecond" />
    </rabbit:listener-container>

    <!--######分隔线######-->
    <!--定义queue -->
    <rabbit:queue name="GP_THIRD_QUEUE_FORM_SPRING" durable="true" auto-delete="false" exclusive="false" declared-by="connectAdmin" />

    <!-- 定义topic exchange，绑定GP_THIRD_QUEUE，注意关键词是pattern -->
    <rabbit:topic-exchange name="GP_TOPIC_EXCHANGE_FORM_SPRING" durable="true" auto-delete="false" declared-by="connectAdmin">
        <rabbit:bindings>
            <rabbit:binding queue="GP_THIRD_QUEUE_FORM_SPRING" pattern="#.Third.#"></rabbit:binding>
        </rabbit:bindings>
    </rabbit:topic-exchange>

    <!--定义rabbit template用于数据的接收和发送 -->
    <rabbit:template id="amqpTemplate2" connection-factory="connectionFactory" exchange="GP_TOPIC_EXCHANGE_FORM_SPRING" />

    <!-- 消息接收者 -->
    <bean id="receiverThird" class="rabbitmq.springapi.consumer.ThirdConsumer"></bean>

    <!-- queue litener 观察 监听模式 当有消息到达时会通知监听在对应的队列上的监听对象 -->
    <rabbit:listener-container connection-factory="connectionFactory">
        <rabbit:listener queues="GP_THIRD_QUEUE_FORM_SPRING" ref="receiverThird" />
    </rabbit:listener-container>

    <!--######分隔线######-->
    <!--定义queue -->
    <rabbit:queue name="GP_FOURTH_QUEUE_FORM_SPRING" durable="true" auto-delete="false" exclusive="false" declared-by="connectAdmin" />

    <!-- 定义fanout exchange，绑定GP_FIRST_QUEUE 和 GP_FOURTH_QUEUE -->
    <rabbit:fanout-exchange name="GP_FANOUT_EXCHANGE_FORM_SPRING" auto-delete="false" durable="true" declared-by="connectAdmin" >
        <rabbit:bindings>
            <rabbit:binding queue="GP_FIRST_QUEUE_FORM_SPRING"></rabbit:binding>
            <rabbit:binding queue="GP_FOURTH_QUEUE_FORM_SPRING"></rabbit:binding>
        </rabbit:bindings>
    </rabbit:fanout-exchange>

    <!-- 消息接收者 -->
    <bean id="receiverFourth" class="rabbitmq.springapi.consumer.FourthConsumer"></bean>

    <!-- queue litener 观察 监听模式 当有消息到达时会通知监听在对应的队列上的监听对象 -->
    <rabbit:listener-container connection-factory="connectionFactory">
        <rabbit:listener queues="GP_FOURTH_QUEUE_FORM_SPRING" ref="receiverFourth" />
    </rabbit:listener-container>
</beans>
```

### 核心

![image-20220721212827272](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207212128342.png)

![image-20220721212840586](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207212128641.png)

## Spring Boot 集成 RabbitMQ

![image-20220721212912820](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207212129904.png)
