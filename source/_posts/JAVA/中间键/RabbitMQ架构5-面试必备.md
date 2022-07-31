---
title: RabbitMQ架构-面试必备
date: 2022-07-26 10:30:37
categories:
- 中间键
- 消息中间键
tag:
top:
keywords:
description:
cover: 'https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207281122220.png'
top_img: 'https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207281122220.png'
---

# 思考：消息可靠性

> 1、在使用 MQ 实现异步通信的过程中，有消息丢了怎么办？ 
>
> 2、MQ 消息重复了怎么办？ 
>
> 3、死信队列满了怎么办？ 
>
> 4、既然消息都产生了，为什么不消费？



首先：保证消息可靠性的同时会降低消息发送的效率，两者不可兼得，但是如果一些业务实时一致性要求不是特别高的场合，可以牺牲一些可靠性来换取 效率。比如发送通知或者记录日志的这种场景，如果用户没有收到通知，不会造成很 大的影响，就不需要严格保证所有的消息都发送成功。如果失败了，只要再次发送就 可以了。

![image-20220728112229122](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207281122220.png)

第一阶段：代表消息从生产者发送到 Broker

生产者把消息发到 Broker 之后，怎么知道自己的消息有没有被 Broker 成功 接收？如果 Broker 不给应答，生产者不断地发送，那有可能是一厢情愿，消息全 部进了黑洞。

第二阶段：代表消息从 Exchange 路由到 Queue

Exchange 是一个绑定列表，它的职责是分发消息。如果它没有办法履行它的 职责怎么办？也就是说，找不到队列或者找不到正确的队列，怎么处理？

第三阶段：代表消息在 Queue 中存储

队列有自己的数据库（Mnesia），它是真正用来存储消息的。如果还没有消 费者来消费，那么消息要一直存储在队列里面。你的信件放在邮局，如果邮局内部 出了问题，比如起火，信件肯定会丢失。怎么保证消息在队列稳定地存储呢？

第四阶段：代表消费者订阅 Queue 并消费消息

队列的特性是什么？FIFO。队列里面的消息是一条一条的投递的，也就是说， 只有上一条消息被消费者接收以后，才能把这一条消息从数据库删掉，继续投递下 一条消息。 或者反过来说，如果消费者不签收，我是不能去派送下一个快件的，总不能丢 在门口就跑吧？



**Broker（快递总部）怎么知道消费者已经接收了消息呢？**



## 消息发送到 RabbitMQ 服务器

> 第一个环节是生产者发送消息到 Broker。

**先来说一下什么情况下会发送消息失 败？**

可能因为网络连接或者 Broker 的问题（比如硬盘故障、硬盘写满了）导致消息发 送失败，生产者不能确定 Broker 有没有正确的接收。

所以设置一个应答机制。需要MQ服务器返回一个应答，生产者接收后代表消息发送成功。

**第一种是 Transaction（事务）模式，第二种 Confirm（确认）模式。**

### Transaction（事务）模式

在创建 channel 的时候，可以把信道设置成事务模式， 然后就可以发布消息给 RabbitMQ 了。如果 channel.txCommit();的方法调用成功， 就说明事务提交成功，则消息一定到达了 RabbitMQ 中。

在事务提交执行之前由于 RabbitMQ 异常崩溃或者其他原因抛出异常，这个 时候我们便可以将其捕获，进而通过执行 channel.txRollback()方法来实现事务回滚。

```java
//JAVAAPI操作
try {
	channel.txSelect();
	// 发送消息
	// String exchange, String routingKey, BasicProperties props, byte[] body
	channel.basicPublish("", QUEUE_NAME, null, (msg).getBytes());
	// int i =1/0;
	channel.txCommit();
	System.out.println("消息发送成功");
} catch (Exception e) {
	channel.txRollback();
	System.out.println("消息已经回滚");
}
```

![image-20220728212441185](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207282124269.png)

**在事务模式里面，只有收到了服务端的 Commit-OK 的指令，才能提交成功。所 以可以解决生产者和服务端确认的问题。但是事务模式有一个缺点，它是阻塞的，一 条消息没有发送完毕，不能发送下一条消息，它会榨干 RabbitMQ 服务器的性能。所 以不建议大家在生产环境使用。**

```java
//Spring Boot 中的设置： 
rabbitTemplate.setChannelTransacted(true);
```

### Confirm（确认）模式 

确认模式有三种，一种是普通确认模式。

#### 普通确认模式

![image-20220728213534009](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207282135111.png)

​	在生产者这边通过调用 channel.confirmSelect()方法将信道设置为 Confirm 模 式，然后发送消息。一旦消息被投递到交换机之后（跟是否路由到队列没有关系）， RabbitMQ 就 会 发 送 一 个 确 认 （ Basic.Ack ） 给 生 产 者 ， 也 就 是 调 用 channel.waitForConfirms()返回 true，这样生产者就知道消息被服务端接收了。

```java
// 开启发送方确认模式
channel.confirmSelect();
channel.basicPublish("", QUEUE_NAME, null, msg.getBytes());
// 普通 Confirm，发送一条，确认一条
if (channel.waitForConfirms()) {
	System.out.println("消息发送成功" );
}

/////////////////////////////////////////////////////////////////////////////////////////////

//批量发送消息
try {
	channel.confirmSelect();
for (int i = 0; i < 5; i++) {
	// 发送消息
	// String exchange, String routingKey, BasicProperties props, byte[] body
	channel.basicPublish("", QUEUE_NAME, null, (msg +"-"+ i).getBytes());
}
	// 批量确认结果，ACK 如果是 Multiple=True，代表 ACK 里面的 Delivery-Tag 之前的消息都被确认了
	// 比如 5 条消息可能只收到 1 个 ACK，也可能收到 2 个（抓包才看得到）
	// 直到所有信息都发布，只要有一个未被 Broker 确认就会 IOException
	channel.waitForConfirmsOrDie();
	System.out.println("消息发送完毕，批量确认成功");
} catch (Exception e) {
	// 发生异常，可能需要对所有消息进行重发
	e.printStackTrace();
}

```

**只要 channel.waitForConfirmsOrDie();方法没有抛出异常，就代表消息都被服务 端接收了。**

批量确认的方式比单条确认的方式效率要高，但是也有两个问题： 

第一个就是批量的数量的确定。对于不同的业务，到底发送多少条消息确认一次？ 数量太少，效率提升不上去。

数量多的话，又会带来另一个问题，比如我们发 1000 条消息才确认一次，如果前面 999 条消息都被服务端接收了，如果第 1000 条消息被 拒绝了，那么前面所有的消息都要重发。



**有没有一种方式，可以一边发送一边确认的呢？**

这个就是异步确认模式。 

异步确认模式需要添加一个 ConfirmListener，并且用一个 SortedSet 来维护一 个批次中没有被确认的消息。

#### 异步确认模式

```java
/**
 * 消息生产者，测试Confirm模式
 */
public class AsyncConfirmProducer {
    private final static String QUEUE_NAME = "ORIGIN_QUEUE";

    public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUri(ResourceUtil.getKey("rabbitmq.uri"));

        // 建立连接
        Connection conn = factory.newConnection();
        // 创建消息通道
        Channel channel = conn.createChannel();

        String msg = "Hello world, Rabbit MQ, Async Confirm";
        // 声明队列（默认交换机AMQP default，Direct）
        // String queue, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object> arguments
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);

        // 用来维护未确认消息的deliveryTag
        final SortedSet<Long> confirmSet = Collections.synchronizedSortedSet(new TreeSet<Long>());

        // 这里不会打印所有响应的ACK；ACK可能有多个，有可能一次确认多条，也有可能一次确认一条
        // 异步监听确认和未确认的消息
        // 如果要重复运行，先停掉之前的生产者，清空队列
        channel.addConfirmListener(new ConfirmListener() {
            public void handleNack(long deliveryTag, boolean multiple) throws IOException {
                System.out.println("Broker未确认消息，标识：" + deliveryTag);
                if (multiple) {
                    // headSet表示后面参数之前的所有元素，全部删除
                    confirmSet.headSet(deliveryTag + 1L).clear();
                } else {
                    confirmSet.remove(deliveryTag);
                }
                // 这里添加重发的方法
            }
            public void handleAck(long deliveryTag, boolean multiple) throws IOException {
                // 如果true表示批量执行了deliveryTag这个值以前（小于deliveryTag的）的所有消息，如果为false的话表示单条确认
                System.out.println(String.format("Broker已确认消息，标识：%d，多个消息：%b", deliveryTag, multiple));
                if (multiple) {
                    // headSet表示后面参数之前的所有元素，全部删除
                    confirmSet.headSet(deliveryTag + 1L).clear();
                } else {
                    // 只移除一个元素
                    confirmSet.remove(deliveryTag);
                }
                System.out.println("未确认的消息:"+confirmSet);
            }
        });

        // 开启发送方确认模式
        channel.confirmSelect();
        for (int i = 0; i < 10; i++) {
            long nextSeqNo = channel.getNextPublishSeqNo();
            // 发送消息
            // String exchange, String routingKey, BasicProperties props, byte[] body
            channel.basicPublish("", QUEUE_NAME, null, (msg +"-"+ i).getBytes());
            confirmSet.add(nextSeqNo);
        }
        System.out.println("所有消息:"+confirmSet);

        // 这里注释掉的原因是如果先关闭了，可能收不到后面的ACK
        //channel.close();
        //conn.close();
    }
}
```

Spring Boot： Confirm 模式是在 Channel 上开启的，RabbitTemplate 对 Channel 进行了封装。

```java
RabbitTemplate rabbitTemplate = context.getBean(RabbitTemplate.class);
rabbitTemplate.setConfirmCallback(new RabbitTemplate.ConfirmCallback(){
    public void confirm(CorrelationData correlationData, boolean ack, String cause) {
        if (ack) {
            System.out.println("消息确认成功");
        } else {
            // nack
            System.out.println("消息确认失败");
        }
    }
});

rabbitTemplate.convertAndSend("GP_BASIC_FANOUT_EXCHANGE", "", "this is a msg");
```

## 消息从交换机路由到队列

在什么情况下，消 息会无法路由到正确的队列？

**有两种方式处理无法路由的消息，一种就是让服务端重发给生产者，一种是 让交换机路由到另一个备份的交换机。**

### 消息回发

```java
channel.addReturnListener(new ReturnListener() {
	public void handleReturn(int replyCode,	String replyText,String exchange,String routingKey,	AMQP.BasicProperties properties,byte[] body)  throws IOException {
	System.out.println("监听器收到了无法路由，被返回的消息======");
	System.out.println("replyText:"+replyText);
	System.out.println("exchange:"+exchange);
	System.out.println("routingKey:"+routingKey);
	System.out.println("message:"+new String(body));
}
});
```

Spring Boot 消息回发的方式：使用 mandatory 参数和 ReturnListener（在 Spring AMQP 中是 ReturnCallback）。

```java
rabbitTemplate.setMandatory(true);
rabbitTemplate.setReturnCallback(new RabbitTemplate.ReturnCallback(){
	public void returnedMessage(Message message,
	int replyCode,
	String replyText,
	String exchange,
	String routingKey){
	System.out.println("回发的消息：");
	System.out.println("replyCode: "+replyCode);
	System.out.println("replyText: "+replyText);
	System.out.println("exchange: "+exchange);
	System.out.println("routingKey: "+routingKey);
}
});
```

### 消息路由到备份交换机的方式

在创建交换机的时候，从属性中指定备份交换机。

```java
Map<String,Object> arguments = new HashMap<String,Object>();
arguments.put("alternate-exchange","ALTERNATE_EXCHANGE"); // 指定交换机的备份交换机
channel.exchangeDeclare("TEST_EXCHANGE","topic", false, false, , arguments);
```

**（注意区别，队列可以指定死信交换机；交换机可以指定备份交换机）**

## 消息在队列存储

### 队列持久化

**没有消费者的话，队列一直存在在数据库中。**

如果 RabbitMQ 的服务或者硬件发生故障，比如系统宕机、重启、关闭等等，可 能会导致内存中的消息丢失，所以我们要把消息本身和元数据（队列、交换机、绑定） 都保存到磁盘。

```java
// 声明队列
// String queue, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object> arguments
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
/*
durable：没有持久化的队列，保存在内存中，服务重启后队列和消息都会消失（钱
和人一起没了）。
autoDelete：没有消费者连接的时候，自动删除。
exclusive：排他性队列的特点是：
1）只对首次声明它的连接（Connection）可见
2）会在其连接断开的时候自动删除。
*/
```

### 交换机持久化

```java
// JAVAAPI声明交换机
// String exchange, String type, boolean durable, boolean autoDelete, Map<String, Object> arguments
channel.exchangeDeclare(EXCHANGE_NAME,"direct",false, false, null);


//JAVASpringBoot API
@Bean("GpExchange")
public DirectExchange exchange() {
// exchangeName, durable, exclusive, autoDelete, Properties
return new DirectExchange("GP_TEST_EXCHANGE", true, false, new HashMap<>());
}
```

### 消息持久化

```java
// 对每条消息设置过期时间
AMQP.BasicProperties properties = new AMQP.BasicProperties.Builder()
	.deliveryMode(2) // 2 代表持久化
	.contentEncoding("UTF-8") // 编码
	.expiration("10000") // TTL，过期时间
	.headers(headers) // 自定义属性
	.priority(5) // 优先级，默认为 5，配合队列的 x-max-priority 属性使用
	.messageId(String.valueOf(UUID.randomUUID()))
	.build();
// 此处两种方式设置消息过期时间的方式都使用了，将以较小的数值为准
// 发送消息
channel.basicPublish("", "TEST_TTL_QUEUE_FROM_JAVAAPI", properties, msg.getBytes());
```

### 集群部署

> 如果只有一个 RabbitMQ 的节点，即使交换机、队列、消息做了持久化，如果服 务崩溃或者硬件发生故障，RabbitMQ 的服务一样是不可用的。
>
> 所以为了提高 MQ 服务的可用性，保障消息的传输，我们需要有多个 RabbitMQ 的节点。

## 消息投递到消费者

**如果消费者收到消息后没来得及处理即发生异常，或者处理过程中发生异常，会 导致④失败。**

服务端应该以某种方式得知消费者对消息的接收情况，并决定是否重新 投递这条消息给其他消费者。

RabbitMQ 提供了消费者的消息确认机制（message acknowledgement），消 费者可以自动或者手动地发送 ACK 给服务端。

**如果没有 ACK 会怎么办？**

没有收到 ACK 的消息，消费者断开连接后，RabbitMQ 会把这条消息发送给其他 消费者。如果没有其他消费者，消费者重启后会重新消费这条消息，重复执行业务逻 辑（如果代码修复好了还好）。

**消费者怎么给 Broker 应答呢？**

有两种方式，一种是自动 ACK，一种是手动 ACK。

1、自动 ACK，这个也是默认的情况。也就是我们没有在消费者处编写 ACK 的 代码，消费者会在收到消息的时候就自动发送 ACK，而不是在方法执行完毕的时候发 送 ACK（并不关心你有没有正常消息）。
2、等消息消费完毕或者方法执行完毕才发送 ACK，需要先把自动 ACK 设置 成手动 ACK。把 autoAck 设置成 false。

```java
// 开始获取消息
// String queue, boolean autoAck, Consumer callback
channel.basicConsume(QUEUE_NAME, false, consumer);
```

这个时候 RabbitMQ 会等待消费者显式地回复 ACK 后才从队列中移去消息。

```java
channel.basicAck(envelope.getDeliveryTag(), true)
```

```java
// 创建消费者，并接收消息
Consumer consumer = new DefaultConsumer(channel) {
    @Override
    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties,
                               byte[] body) throws IOException {
        String msg = new String(body, "UTF-8");
        System.out.println("Received message : '" + msg + "'");

        if (msg.contains("拒收")){
            // 拒绝消息
            // requeue：是否重新入队列，true：是；false：直接丢弃，相当于告诉队列可以直接删除掉
            // TODO 如果只有这一个消费者，requeue 为true 的时候会造成消息重复消费
            channel.basicReject(envelope.getDeliveryTag(), false);
        } else if (msg.contains("异常")){
            // 批量拒绝
            // requeue：是否重新入队列
            // TODO 如果只有这一个消费者，requeue 为true 的时候会造成消息重复消费
            channel.basicNack(envelope.getDeliveryTag(), true, false);
        } else {
            // 手工应答
            // 如果不应答，队列中的消息会一直存在，重新连接的时候会重复消费
            channel.basicAck(envelope.getDeliveryTag(), true);
        }
    }
};
```

在 Spring Boot 中： application.properties

```
spring.rabbitmq.listener.direct.acknowledge-mode=manual
spring.rabbitmq.listener.simple.acknowledge-mode=manual
```

SimpleRabbitListenerContainer或者SimpleRabbitListenerContainerFactory

```java
factory.setAcknowledgeMode(AcknowledgeMode.MANUAL);
/*
注意这三个值的区别： 
NONE：自动 ACK 
MANUAL： 手动 ACK 
AUTO：如果方法未抛出异常，则发送 ack。
*/
```

如果方法抛出异常，并且不是 AmqpRejectAndDontRequeueException 则发送 nack，并且重新入队列。

如果抛出 异常时 AmqpRejectAndDontRequeueException 则发送 nack 不会重新入队列。

**消费者又怎么调用 ACK，或者说怎么获得 Channel 参数呢？**

```java
@RabbitHandler
public void process(String msg, Channel channel, Message message) throws IOException {
	System.out.println(" second queue received msg : " + msg);
	channel.basicAck(message.getMessageProperties().getDeliveryTag(), false);
}
```

**如果消费出了问题，确实是不能发送 ACK 告诉服务端成功消费了怎么办？**

有拒绝消息的指令，而且还可以让消息重新入队给其他消费者消费。 

如果消息无法处理或者消费失败，也有两种拒绝的方式，Basic.Reject()拒绝单条， Basic.Nack()批量拒绝。

```java
// 创建消费者，并接收消息
Consumer consumer = new DefaultConsumer(channel) {
    @Override
    public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties,
                               byte[] body) throws IOException {
        String msg = new String(body, "UTF-8");
        System.out.println("Received message : '" + msg + "'");

        if (msg.contains("拒收")){
            // 拒绝消息
            // requeue：是否重新入队列，true：是；false：直接丢弃，相当于告诉队列可以直接删除掉
            // TODO 如果只有这一个消费者，requeue 为true 的时候会造成消息重复消费
            channel.basicReject(envelope.getDeliveryTag(), false);
        } else if (msg.contains("异常")){
            // 批量拒绝
            // requeue：是否重新入队列
            // TODO 如果只有这一个消费者，requeue 为true 的时候会造成消息重复消费
            channel.basicNack(envelope.getDeliveryTag(), true, false);
        } else {
            // 手工应答
            // 如果不应答，队列中的消息会一直存在，重新连接的时候会重复消费
            channel.basicAck(envelope.getDeliveryTag(), true);
        }
    }
};
```

如果 requeue 参数设置为 true，可以把这条消息重新存入队列，以便发给下一个 消费者

**（当然，只有一个消费者的时候，这种方式可能会出现无限循环重复消费的情 况。可以投递到新的队列中，或者只打印异常日志）。**

## 总结可靠性

从生产者到 Broker、交换机到队列，队列本身，队列到消费者，我们都有相应的 方法知道消费有没有正常流转，或者说当消息没有正常流转的时候采取相关措施。

**服务端收到了 ACK 或者 NACK，生产者会知道吗？**

即使消费者没有接收到 消息，或者消费时出现异常，生产者也是完全不知情的。这个是符合解耦思想的，不然你用 MQ 干嘛？

# 思考：一致性

**如果现在为了保证一致性，生产者必须知道消费者有没有成功消费，怎么办？**

生产者最终确定消费者有没有消费成功的两种方式： 

1） 消费者收到消息，处理完毕后，调用生产者的 API（思考：是否破坏解耦？） 

2） 消费者收到消息，处理完毕后，发送一条响应消息给生产者。

> 支付系统，除了通知类的消息，所有的消息都需要有回执。

![image-20220731080057369](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207310800456.png)

## 消费者回调

对于金融场景下的消息可靠性保证， 是很有用的。

**调用生产者 API** 

例如：提单系统给其他系统发送了保险消息后（通知通知！发生了一笔保险）， 其他系统必须在处理完消息后调用提单系统提供的 API，来修改提单系统中这笔数 据的状态。只要 API 没有被调用，数据状态没有被修改，提单系统就认为下游系 统没有收到这条消息。

**发送响应消息给生产者** 

例如：商业银行与人民银行二代支付通信（使用 IBM MQ），无论是人行收 到了商业银行的消息，还是商业银行收到了人行的消息，都必须发送一条响应消 息（叫做回执报文）。

## 补偿机制

**如果生产者的 API 就是没有被调用，也没有收到消费者的响应消息，怎么办？**

可能是消费者处理时间太长或者网络超时。

生产者与消费者之间应该约定一个超时时间，对于超出这个时间没有得到响应的 消息，才确定为消费失败。

比如 5 分钟。

如果确定失败就要重发消息。

### 谁来重发？

首先消息大部分都是异步的，业务员发送完就不再参与了。

可以先创建一个定时任务，比如每 30 秒跑一次，找到业务表里面的这条业务状态是中 间状态的记录，查询出来，构建为 MQ 消息，重新发送。 

也可以单独设计设计一张消息表，把本系统所以发送出去的消息全部异步地登记 起来，找出状态是未回复的消息发送

但是会消耗性能和系统存储资源。

### 隔多久重发一次？

时间由定时任务的执行时间决定。也可以设置衰减机制

### 一共重发几次？

重发消息务必要控制次数，比如设置成 3 次。 这个要在消息表里面记录次数来实现，发一次就加 1。

### 重发什么内容？

**重发，是否发送一模一样的消息？** 

参考： 

ATM 机上运行的系统叫 C 端（ATMC），银行的前置系统或者渠道系统叫 P 端 （ATMP），它接收 ATMC 的消息，再转发给卡系统或者核心系统。 

1）如果客户存款，没有收到核心系统的应答。怎么处理？ 因为不知道有没有记账成功，不能给客户吐钞，否则会造成银行短款。因为已经 吞钞了，所以要保证成功。最多发送 3 次存款确认报文； 

2）如果客户取款，ATMC 未得到核心系统的应答时，怎么处理？ 因为没有吐钞，所以要保证失败。最多发送 3 次存款冲正报文

## 消息幂等性

什么叫消息幂等性？

用户对于同一操作发起的一次或者多次请求，最后的结果都 是相同的，这就是幂等性



假设消费者状态是正常的，每一条消息都可以正常处理。只是在响应或者调用 API 的时候出了问题，会不会出现消息的重复处理？

例如：存款 1000 元，ATMC 重发了 3 次存款消息，核心系统一共处理了 4 次，账户余额增加了 4000 元。

**但是MQ没有机制知道什么是重复的消息**

> 消息出现重复可能会有三个原因： 

1、生产者的问题，环节①重复发送消息，比如在开启了 Confirm 模式但未收 到确认，消费者重复投递。

 2、消费环节④出了问题，由于消费者未发送 ACK 或者其他原因，消息重复消 费。 

3、生产者代码或者网络问题。

**对于重复发送的消息，可以对每一条消息生成一个唯一的业务 ID，通过日志或者 消息落库来做重复控制。**

**在金融系统中有一个叫流水号的东西。不管你在柜面汇款，还是 ATM 取款， 或者信用卡消费，都会有一个唯一的序号。通过这个序号就可以找到唯一的一笔消息。**银行的重账控制环节，对于进来的每一笔交易，第一件要做的事情就是查 询是否重复。

业务要素一致（付款人 ID、商户 ID、交易类型、金额、交易地 点、交易时间）

![image-20220731082016819](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207310820874.png)

## 最终一致

例如：

在金融系统中，会有双方对账或者多方对账的操作。

金融系统会约定一个标准，比如 ATM 跟核心系统对账，肯定是以核心系统的账务 为准。ATMC 获取到核心系统的对账文件，然后解析，登记成数据，然后跟自己记录 的流水比较，找出核心系统有 ATM 没有，或者 ATM 有核心系统没有，或者两边都有 但是金额不一致的数据。

对账之后，我们再手工平账。比如取款记了账但是没吐钞的，做一笔冲正。存款 吞了钞没记账的，要么把钱退给客户，要么补一笔账。

## 消息的顺序性

消息的顺序性指的是消费者消费消息的顺序跟生产者生产消息的顺序是一致的。

**在 RabbitMQ 中，一个队列有多个消费者时，由于不同的消费者消费消息的速度 是不一样的，顺序无法保证。只有一个队列仅有一个消费者的情况才能保证顺序消费 （不同的业务消息发送到不同的专用的队列）。**

![image-20220731082357069](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207310823141.png)

# 面试题

## 消息队列的作用与使用场景？

**消除高并发访问高峰，加快网站的响应速度**

异步处理，应用解耦，流量削锋和消息通讯四个场景

[消息队列使用的四种场景介绍，有图有解析，一看就懂 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/55712984)

异步处理：（注册用户后，发送注册短信和邮箱可以异步发送）

传统的做法有两种 1.串行的方式；2.并行方式
