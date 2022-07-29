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

# 思考

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