---
title: RabbitMQ架构-API
date: 2022-07-22 08:30:37
categories:
- 中间键
- 消息中间键
tag:
top:
keywords:
description:
cover: 'https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207221532969.png'
top_img: 'https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207221532969.png'
---

# API介绍

## JAVA基础API

![image-20220721173548923](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207221353827.png)

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

![image-20220721212538174](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207221353787.png)

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

![image-20220721212827272](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207221353758.png)

![image-20220721212840586](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207221353715.png)

## Spring Boot 集成 RabbitMQ

![image-20220721212912820](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207221355791.png)

```
<dependencies>
   <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-amqp</artifactId>
   </dependency>

   <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-test</artifactId>
      <scope>test</scope>
   </dependency>
       <dependency>
           <groupId>junit</groupId>
           <artifactId>junit</artifactId>
       </dependency>
   <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-test</artifactId>
   </dependency>
   <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-test</artifactId>
      <version>5.1.3.RELEASE</version>
   </dependency>

   <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-core</artifactId>
      <version>2.9.0</version>
   </dependency>
   <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-databind</artifactId>
      <version>2.9.0</version>
   </dependency>
   <dependency>
      <groupId>com.fasterxml.jackson.core</groupId>
      <artifactId>jackson-annotations</artifactId>
      <version>2.9.0</version>
   </dependency>

</dependencies>
```



```java
package com.gupaoedu.vip.mq.rabbit.springbootapi.basic.config;

import org.springframework.amqp.core.*;
import org.springframework.amqp.rabbit.connection.CachingConnectionFactory;
import org.springframework.amqp.rabbit.connection.ConnectionFactory;
import org.springframework.amqp.rabbit.core.RabbitAdmin;
import org.springframework.amqp.rabbit.core.RabbitTemplate;
import org.springframework.amqp.rabbit.listener.SimpleMessageListenerContainer;
import org.springframework.amqp.support.ConsumerTagStrategy;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.HashMap;
import java.util.Map;


@Configuration
public class RabbitConfig {
    /**
     * 都可以使用缺省对象
     * @return
     * @throws Exception
     */
    @Bean
    public ConnectionFactory connectionFactory() throws Exception {
        CachingConnectionFactory cachingConnectionFactory = new CachingConnectionFactory();
        cachingConnectionFactory.setUri("amqp://guest:guest@192.168.100.149:5601");
        return cachingConnectionFactory;
    }

    @Bean
    public RabbitAdmin amqpAdmin(ConnectionFactory connectionFactory) {
        RabbitAdmin admin = new RabbitAdmin(connectionFactory);
        admin.setAutoStartup(true);
        return admin;
    }

    @Bean
    public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
        return new RabbitTemplate(connectionFactory);
    }

    @Bean
    public SimpleMessageListenerContainer container(ConnectionFactory connectionFactory) {
        SimpleMessageListenerContainer container = new SimpleMessageListenerContainer(connectionFactory);
        container.setConsumerTagStrategy(new ConsumerTagStrategy() {
            public String createConsumerTag(String queue) {
                return null;
            }
        });
        return container;
    }

    // 两个交换机
    @Bean("topicExchange")
    public TopicExchange getTopicExchange(){
        return new TopicExchange("GP_BASIC_TOPIC_EXCHANGE_FORM_SPRINGBOOT");
    }

    @Bean("fanoutExchange")
    public FanoutExchange getFanoutExchange(){
        return  new FanoutExchange("GP_BASIC_FANOUT_EXCHANGE_FORM_SPRINGBOOT");
    }

    // 三个队列
    @Bean("firstQueue")
    public Queue getFirstQueue(){
        Map<String, Object> args = new HashMap<String, Object>();
        args.put("x-message-ttl",6000);
        Queue queue = new Queue("GP_BASIC_FIRST_QUEUE_FORM_SPRINGBOOT", false, false, true, args);
        return queue;
    }

    @Bean("secondQueue")
    public Queue getSecondQueue(){
        return new Queue("GP_BASIC_SECOND_QUEUE_FORM_SPRINGBOOT");
    }

    @Bean("thirdQueue")
    public Queue getThirdQueue(){
        return new Queue("GP_BASIC_THIRD_QUEUE_FORM_SPRINGBOOT");
    }

    // 两个绑定
    @Bean
    public Binding bindSecond(@Qualifier("secondQueue") Queue queue, @Qualifier("topicExchange") TopicExchange exchange){
        return BindingBuilder.bind(queue).to(exchange).with("#.gupao.#");
    }

    @Bean
    public Binding bindThird(@Qualifier("thirdQueue") Queue queue, @Qualifier("fanoutExchange") FanoutExchange exchange){
        return BindingBuilder.bind(queue).to(exchange);
    }
    
}
```

```java
@SpringBootApplication(scanBasePackages = "com.gupaoedu.vip.mq.rabbit.springbootapi.basic")
public class BasicSender {

    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(BasicSender.class);
//        RabbitAdmin rabbitAdmin = context.getBean(RabbitAdmin.class);
        RabbitTemplate rabbitTemplate = context.getBean(RabbitTemplate.class);

        rabbitTemplate.convertAndSend("","GP_BASIC_FIRST_QUEUE_FORM_SPRINGBOOT","-------- a direct msg");
        rabbitTemplate.convertAndSend("GP_BASIC_TOPIC_EXCHANGE_FORM_SPRINGBOOT","shanghai.gupao.teacher","-------- a topic msg : shanghai.gupao.teacher");
        rabbitTemplate.convertAndSend("GP_BASIC_TOPIC_EXCHANGE_FORM_SPRINGBOOT","changsha.gupao.student","-------- a topic msg : changsha.gupao.student");
        rabbitTemplate.convertAndSend("GP_BASIC_FANOUT_EXCHANGE_FORM_SPRINGBOOT","","-------- a fanout msg");

    }
}


@Component
@RabbitListener(queues = "GP_BASIC_FIRST_QUEUE_FORM_SPRINGBOOT")
public class FirstConsumer {

    @RabbitHandler(isDefault = true)
    public void process(String msg, Channel channel) throws IOException {
//        channel.basicAck(deliveryTag, true);
        System.out.println(" first queue received msg : " + msg);
    }
}

@Component
@RabbitListener(queues = "GP_BASIC_SECOND_QUEUE_FORM_SPRINGBOOT")
public class SecondConsumer {

    @RabbitHandler(isDefault = true)
    public void process(String msg){
        System.out.println(" second queue received msg : " + msg);
    }
}
```

## 理解

>首先配置好rabbitmq的Config配置类，包括ConnectionFactory，SimpleMessageListenerContainer（监听器），RabbitTemplate。还有队列，交换机，绑定队列和交换机。
>
>生产者：通过容器得到rabbitTemplate，convertAndSend消息，直连不需要交换机；主题需要指定交换机，和路由key；广播不需要队列/路由key。
>
>消费者：添加监听队列，使用的就是push模式。