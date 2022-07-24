---
title: RabbitMQ架构-流控-延时-高可用操作
date: 2022-07-22 21:30:37
categories:
- 中间键
- 消息中间键
tag:
top:
keywords:
description:
cover: 'https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207231135665.png'
top_img: 'https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207231135665.png'
---

# 订单延时关闭问题

假设有一个业务场景：超过 30 分钟未付款的订单自动关闭，这个功能应该怎么实 现？怎么实现在指定的时候之后消息才发给消费者呢?

 

思路：发一条跟订单相关的消息，30 分钟以后被消费，在消费者的代码中查询订 单数据，如果支付状态是未付款，就关闭订单。

RabbitMQ 本身不支持延迟投递，总的来说有 2 种实现方案： 

1、 先存储到数据库，用定时任务扫描 

2、 利用 RabbitMQ 的死信队列（Dead Letter Queue）实现 定时任务比较容易实现，比如每隔 1 分钟扫描一次，查出 30 分钟之前未付款的订 单，把状态改成关闭。但是如果瞬间要处理的数据量过大，比如 10 万条，把这些全部 的数据查询到内存中逐条处理，也会给服务器带来很大的压力，影响正常业务的运行。

**死信队列怎么实现呢**

首先了解消息存活时间TTL(Time To Live）和死信。

## 消息存活时间TTL(Time To Live）

### Queue 属性设置 

首先，队列有一个消息过期属性。就像丰巢超过 24 小时就收费一样，通过设置这 个属性，超过了指定时间的消息将会被丢弃。 

这个属性叫：x-message-ttl 

所有队列中的消息超过时间未被消费时，都会过期。不管是谁的包裹都一视同仁。

```java
@Bean("ttlQueue")
public Queue queue() {
    Map<String, Object> map = new HashMap<String, Object>();
    map.put("x-message-ttl", 11000); // 队列中的消息未被消费11秒后过期
    // map.put("x-expire", 30000); // 队列30秒没有使用以后会被删除
    return new Queue("GP_TTL_QUEUE", true, false, false, map);
}
```

### Message 属性设置

在发送消息的时候通过 MessageProperties 指定消息属性

```java
MessageProperties messageProperties = new MessageProperties();
messageProperties.setExpiration("4000"); // 消息的过期属性，单位ms
messageProperties.setDeliveryMode(MessageDeliveryMode.PERSISTENT);
Message message = new Message("这条消息4秒后过期".getBytes(), messageProperties);
rabbitTemplate.send("GP_TTL_EXCHANGE", "gupao.ttl", message);

// 随队列的过期属性过期，单位ms
rabbitTemplate.convertAndSend("GP_TTL_EXCHANGE", "gupao.ttl", "这条消息");
```

问题：如果队列 TTL 是 6 秒钟过期，msg TTL 是 10 秒钟过期，这个消息会在什 么时候被丢弃？ 

如果同时指定了 Message TTL 和 Queue TTL，则小的那个时间生效。 

有了过期时间还不够，这个消息不能直接丢弃，不然就没办法消费了。最好是丢 到一个容器里面，这样就可以实现延迟消费了

## 死信（Dead Letter）

消息过期以后，如果没有任何配置，是会直接丢弃的。我们可以通过配置让这样 的消息变成死信（Dead Letter），在别的地方存储

### 死信交换机 DLX 与死信队列 DLQ

队列在创建的时候可以指定一个死信交换机 DLX（Dead Letter Exchange）。

死 信交换机绑定的队列被称为死信队列 DLQ（Dead Letter Queue），DLX 实际上也是 普通的交换机，DLQ 也是普通的队列

![image-20220723122058705](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207231220777.png)

也就是说，如果消息过期了，队列指定了 DLX，就会发送到 DLX。如果 DLX 绑定 了 DLQ，就会路由到 DLQ。路由到 DLQ 之后，我们就可以消费了。

### 死信队列的实现方案

1、 声 明 原 交 换 机 （ GP_ORI_USE_EXCHANGE ） 、 原 队 列 （ GP_ORI_USE_QUEUE ） ， 相 互 绑 定 。 指 定 原 队 列 的 死 信 交 换 机 （GP_DEAD_LETTER_EXCHANGE）。 

2、声明死信交换机（GP_DEAD_LETTER_EXCHANGE ）、死信队列 （GP_DEAD_LETTER_QUEUE），并且通过"#"绑定，代表无条件路由 

3、最终消费者监听死信队列，在这里面实现检查订单状态逻辑。 

4、生产者发送消息测试，设置消息 10 秒过期。

#### JAVA-API

```java
/**
 * 消息消费者，由于消费的代码被注释掉了，
 * 10秒钟后，消息会从正常队列 GP_ORI_USE_QUEUE 到达死信交换机 GP_DEAD_LETTER_EXCHANGE ，然后路由到死信队列 GP_DEAD_LETTER_QUEUE
 *
 */
public class DlxConsumer {

    public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUri(ResourceUtil.getKey("rabbitmq.uri"));
        // 建立连接
        Connection conn = factory.newConnection();
        // 创建消息通道
        Channel channel = conn.createChannel();

        // 指定队列的死信交换机
        Map<String,Object> arguments = new HashMap<String,Object>();
        arguments.put("x-dead-letter-exchange","GP_DEAD_LETTER_EXCHANGE");
        // arguments.put("x-expires",9000L); // 设置队列的TTL
        // arguments.put("x-max-length", 4); // 如果设置了队列的最大长度，超过长度时，先入队的消息会被发送到DLX

        // 声明队列（默认交换机AMQP default，Direct）
        // String queue, boolean durable, boolean exclusive, boolean autoDelete, Map<String, Object> arguments
        channel.queueDeclare("GP_ORI_USE_QUEUE", false, false, false, arguments);

        // 声明死信交换机
        channel.exchangeDeclare("GP_DEAD_LETTER_EXCHANGE","topic", false, false, false, null);
        // 声明死信队列
        channel.queueDeclare("GP_DEAD_LETTER_QUEUE", false, false, false, null);
        // 绑定，此处 Dead letter routing key 设置为 #
        channel.queueBind("GP_DEAD_LETTER_QUEUE","GP_DEAD_LETTER_EXCHANGE","#");
        System.out.println(" Waiting for message....");

        // 创建消费者
        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties,
                                       byte[] body) throws IOException {
                String msg = new String(body, "UTF-8");
                System.out.println("Received message : '" + msg + "'");
            }
        };

        // 开始获取消息
        // String queue, boolean autoAck, Consumer callback
        channel.basicConsume("GP_DEAD_LETTER_QUEUE", true, consumer);
    }
}
```



```java
/**
 * 消息生产者，通过TTL测试死信队列
 */
public class DlxProducer {

    public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUri(ResourceUtil.getKey("rabbitmq.uri"));

        // 建立连接
        Connection conn = factory.newConnection();
        // 创建消息通道
        Channel channel = conn.createChannel();

        String msg = "Hello world, Rabbit MQ, DLX MSG";

        // 设置属性，消息10秒钟过期
        AMQP.BasicProperties properties = new AMQP.BasicProperties.Builder()
                .deliveryMode(2) // 持久化消息
                .contentEncoding("UTF-8")
                .expiration("10000") // TTL
                .build();

        // 发送消息
        channel.basicPublish("", "GP_ORI_USE_QUEUE", properties, msg.getBytes());

        channel.close();
        conn.close();
    }
}
```

#### SpringBoot-API

```java
/**
 * 队列的死信交换机
 * @return
 */
@Bean("deatLetterExchange")
public TopicExchange deadLetterExchange() {
    return new TopicExchange("GP_DEAD_LETTER_EXCHANGE", true, false, new HashMap<>());
}

@Bean("deatLetterQueue")
public Queue deadLetterQueue() {
    return new Queue("GP_DEAD_LETTER_QUEUE", true, false, false, new HashMap<>());
}
```

```java
@ComponentScan(basePackages = "com.gupaoedu.vip.mq.rabbit.springbootapi.dlx.ttl")
public class DlxSender {

    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(DlxSender.class);
        RabbitAdmin rabbitAdmin = context.getBean(RabbitAdmin.class);
        RabbitTemplate rabbitTemplate = context.getBean(RabbitTemplate.class);

        // 随队列的过期属性过期，单位ms
        rabbitTemplate.convertAndSend("GP_ORI_USE_EXCHANGE", "gupao.ori.use", "测试死信消息");

    }
}
```

## 死信消息流转原理

![image-20220723113558547](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207231135665.png)

### 总结

> 利用消息的过期时间，过期之后投递到 DLX，路由到 DLQ，监听 DLQ， 实现了延迟队列。 
>
> 消息的流转流程： 生产者——原交换机——原队列（超过 TTL 之后）——死信交换机——死信队列 ——最终消费者

## 延迟队列的其他实现方案

死信队列实现延时消息的缺点：

1） 如果统一用队列来设置消息的 TTL，当梯度非常多的情况下，比如 1 分钟，2 分钟，5 分钟，10 分钟，20 分钟，30 分钟……需要创建很多交换机和队列来路由消息。 

2） 如果单独设置消息的 TTL，则可能会造成队列中的消息阻塞——前一条消息 没有出队（没有被消费），后面的消息无法投递（比如第一条消息过期 TTL 是 30min， 第二条消息 TTL 是 10min。10 分钟后，即使第二条消息应该投递了，但是由于第一条 消息还未出队，所以无法投递）。

 3） 可能存在一定的时间误差。

解决方法：

在 RabbitMQ 3.5.7 及 以 后 的 版 本 提 供 了 一 个 插 件 （ rabbitmq-delayed-message-exchange ） 来 实 现 延 时 队 列 功 能 （ Linux 和 Windows 都可用）。同时插件依赖 Erlang/OPT 18.0 及以上。

https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/releases

下载的ez文件放入：

```java
cd /usr/lib/rabbitmq/lib/rabbitmq_server-3.8.11/plugins
```



```
//下载插件
wget https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/releases/download/3.8.9/
rabbitmq_delayed_message_exchange-3.8.9-0199d11c.ez

//启用 rabbitmq_delayed_message_exchange
rabbitmq-plugins enable rabbitmq_delayed_message_exchange

//重启服务
service rabbitmq-server restart
或者
rabbitmq-server restar
```

### 插件使用

通 过 声 明 一 个 x-delayed-message 类 型 的 Exchange 来 使 用 delayed-messaging 特 性 。

 x-delayed-message 是 插 件 提 供 的 类 型 ， 并 不 是 RabbitMQ 本身的（区别于 direct、topic、fanout、headers）

![image-20220723172933387](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207231729472.png)

### JAVA-API

```java
/**
 *  使用延时插件实现的消息投递-消费者
 *  必须要在服务端安装rabbitmq-delayed-message-exchange插件，安装步骤见README.MD
 *  先启动消费者
 */
public class DelayPluginConsumer {

    public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUri("amqp://guest:guest@127.0.0.1:5672");
        // 建立连接
        Connection conn = factory.newConnection();
        // 创建消息通道
        Channel channel = conn.createChannel();

        // 声明x-delayed-message类型的exchange
        Map<String, Object> argss = new HashMap<String, Object>();
        argss.put("x-delayed-type", "direct");
        channel.exchangeDeclare("DELAY_EXCHANGE", "x-delayed-message", false,
                false, argss);

        // 声明队列
        channel.queueDeclare("DELAY_QUEUE", false,false,false,null);

        // 绑定交换机与队列
        channel.queueBind("DELAY_QUEUE", "DELAY_EXCHANGE", "DELAY_KEY");

        // 创建消费者
        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties,
                                       byte[] body) throws IOException {
                String msg = new String(body, "UTF-8");
                SimpleDateFormat sf=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss.SSS");
                System.out.println("收到消息：[" + msg + "]\n接收时间：" +sf.format(new Date()));
            }
        };

        // 开始获取消息
        // String queue, boolean autoAck, Consumer callback
        channel.basicConsume("DELAY_QUEUE", true, consumer);
    }
}
```

```java
/**
 *  使用延时插件实现的消息投递-生产者
 *  必须要在服务端安装rabbitmq-delayed-message-exchange插件，安装步骤见README.MD
 *  先启动消费者
 */
public class DelayPluginProducer {

    public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUri("amqp://guest:guest@127.0.0.1:5672");

        // 建立连接
        Connection conn = factory.newConnection();
        // 创建消息通道
        Channel channel = conn.createChannel();

        // 延时投递，比如延时1分钟
        Date now = new Date();
        Calendar calendar = Calendar.getInstance();
        calendar.add(Calendar.SECOND, +10);//
        Date delayTime = calendar.getTime();

        // 定时投递，把这个值替换delayTime即可
        // Date exactDealyTime = new Date("2019/01/14,22:30:00");

        SimpleDateFormat sf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss.SSS");
        String msg = "发送时间：" + sf.format(now) + "，投递时间：" + sf.format(delayTime);

        // 延迟的间隔时间，目标时刻减去当前时刻
        Map<String, Object> headers = new HashMap<String, Object>();
        headers.put("x-delay", delayTime.getTime() - now.getTime());

        AMQP.BasicProperties.Builder props = new AMQP.BasicProperties.Builder()
                .headers(headers);
        channel.basicPublish("DELAY_EXCHANGE", "DELAY_KEY", props.build(),
                msg.getBytes());

        channel.close();
        conn.close();
    }
}
```

### SpringBoot-API

```java
@Configuration
public class DelayPluginConfig {
    @Bean
    public ConnectionFactory connectionFactory() throws Exception {
        CachingConnectionFactory cachingConnectionFactory = new CachingConnectionFactory();
        cachingConnectionFactory.setUri("amqp://guest:guest@192.168.100.149:5601");
        return cachingConnectionFactory;
    }

    @Bean
    public RabbitAdmin rabbitAdmin(ConnectionFactory connectionFactory) {
        return new RabbitAdmin(connectionFactory);
    }

    @Bean
    public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
        return new RabbitTemplate(connectionFactory);
    }

    @Bean("delayExchange")
    public TopicExchange exchange() {
        Map<String, Object> argss = new HashMap<String, Object>();
        argss.put("x-delayed-type", "direct");
        return new TopicExchange("GP_DELAY_EXCHANGE", true, false, argss);
    }

    @Bean("delayQueue")
    public Queue deadLetterQueue() {
        return new Queue("GP_DELAY_QUEUE", true, false, false, new HashMap<>());
    }

    @Bean
    public Binding bindingDead(@Qualifier("delayQueue") Queue queue, @Qualifier("delayExchange") TopicExchange exchange) {
        return BindingBuilder.bind(queue).to(exchange).with("#"); // 无条件路由
    }

}
```

```java
public class DelayPluginConsumer {
    public static void main(String[] args) throws Exception {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setUri("amqp://guest:guest@192.168.100.149:5601");
        // 建立连接
        Connection conn = factory.newConnection();
        // 创建消息通道
        Channel channel = conn.createChannel();

        // 创建消费者
        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties,
                                       byte[] body) throws IOException {
                String msg = new String(body, "UTF-8");
                SimpleDateFormat sf=new SimpleDateFormat("yyyy-MM-dd HH:mm:ss.SSS");
                System.out.println("收到消息：[" + msg + "]\n接收时间：" +sf.format(new Date()));
            }
        };

        // 开始获取消息
        // String queue, boolean autoAck, Consumer callback
        channel.basicConsume("GP_DELAY_QUEUE", true, consumer);
    }
}
```

```java
/**
 * 延时消息插件，去管控台队列看有无收到消息
 * 不能在本地测试，必须发送消息到安装了插件的服务端
 */
@ComponentScan(basePackages = "com.gupaoedu.vip.mq.rabbit.springbootapi.dlx.delayplugin")
public class DelayPluginProducer {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(DelayPluginProducer.class);
        RabbitAdmin rabbitAdmin = context.getBean(RabbitAdmin.class);
        RabbitTemplate rabbitTemplate = context.getBean(RabbitTemplate.class);

        // 延时投递，比如延时4秒
        Date now = new Date();
        Calendar calendar = Calendar.getInstance();
        calendar.add(Calendar.SECOND, +4);
        Date delayTime = calendar.getTime();

        // 定时投递，把这个值替换delayTime即可
        // Date exactDealyTime = new Date("2019/06/24,22:30:00");
        SimpleDateFormat sf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss.SSS");
        String msg = "延时插件测试消息，发送时间：" + sf.format(now) + "，理论路由时间：" + sf.format(delayTime);

        MessageProperties messageProperties = new MessageProperties();
        // 延迟的间隔时间，目标时刻减去当前时刻
        messageProperties.setHeader("x-delay", delayTime.getTime() - now.getTime());
        Message message = new Message(msg.getBytes(), messageProperties);

        // 不能在本地测试，必须发送消息到安装了插件的服务端
        rabbitTemplate.send("GP_DELAY_EXCHANGE", "#", message);

    }
}
```



# 服务端流控（Flow Control）

## 队列长度

队列有两个控制长度的属性： 

x-max-length：队列中最大存储最大消息数，超过这个数量，队头的消息会被 丢弃。 

x-max-length-bytes：队列中存储的最大消息容量（单位 bytes），超过这个 容量，队头的消息会被丢弃

![image-20220724185555349](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207241855452.png)

注意的是，设置队列长度只在消息堆积的情况下有意义，而且会删除先入队 的消息，不能真正地实现服务端限流。

## 内存控制

RabbitMQ 会在内存超过阈值时告警，默认为 40% 以上内存时，MQ 会主动抛出一个内存警告并阻塞所有连接（Connections）。 可以通过调整阈值来控制消息的流量。

```
rabbitmqctl set_vm_memory_high_watermark 0.6
//如果设置成 0，则所有的消息都不能发布。
```

## 磁盘控制

另一种方式是通过磁盘来控制消息的发布。当磁盘剩余可用空间低于指定的值时 （默认 50MB），触发流控措施。 

例如：指定为磁盘的 30%或者 2GB： https://www.rabbitmq.com/configure.html 

```
disk_free_limit.relative = 3.0 

disk_free_limit.absolute = 2G
```

> 还有一种情况，虽然 Broker 消息存储得过来，但是在 push 模型下（consume， 有消息就消费），消费者消费不过来了，这个时候也要对流量进行控制。

# 消费端限流

https://www.rabbitmq.com/consumer-prefetch.html 

​	默认情况下，如果不进行配置，RabbitMQ 会尽可能快速地把队列中的消息发送 到消费者。因为消费者会在本地缓存消息，如果消息数量过多，可能会导致 OOM 或 者影响其他进程的正常运行

​	在消费者处理消息的能力有限，例如消费者数量太少，或者单条消息的处理时间 过长的情况下，如果我们希望在一定数量的消息消费完之前，不再推送消息过来，就 要用到消费端的流量限制措施。

​	可以基于 Consumer 或者 channel 设置 prefetch count 的值，含义为 Consumer 端的最大的 unacked messages 数目。当超过这个数值的消息未被确认，RabbitMQ 会停止投递新的消息给该消费者。

```java
channel.basicQos(2); 
// 如果超过 2 条消息没有发送 ACK，当前消费者不再接受队列消息
channel.basicConsume(QUEUE_NAME, false, consumer);
//启动两个消费者，其中一个 Consumer2 消费很慢，qos 设置为 2，最多一次给它发两条消息，其他的消息都被 Consumer1 接收了
```

# 高可用集群实现

## 为什么要做集群

集群主要用于实现高可用与负载均衡。 

高可用：如果集群中的某些 MQ 服务器不可用，客户端还可以连接到其他 MQ 服 务器。不至于影响业务。 

负载均衡：在高并发的场景下，单台 MQ 服务器能处理的消息有限，可以分发给 多台 MQ 服务器。减少消息延迟。

## RabbitMQ 如何支持集群

​	应用做集群，需要面对数据同步和通信的问题。

​	因为 Erlang 天生具备分布式的特 性，所以 RabbitMQ 天然支持集群，不需要通过引入 ZK 来实现数据同步。 

​	RabbitMQ 通过.erlang.cookie（默认路径：/var/lib/rabbitmq/）来验证身份， 需要在所有节点上保持一致。

服务的端口是 5672，UI 的端口是 15672，集群的端口是 25672。 集群通过 25672 端口两两通信，需要开放防火墙的端口。 需要注意的是，RabbitMQ 集群无法搭建在广域网上，除非使用 federation 或者 shovel 等插件（没这个必要，在同一个机房做集群）。

## RabbitMQ 节点类型

集群有两种节点类型，一种是磁盘节点（Disc Node），一种是内存节点（RAM Node）。 磁盘节点：将元数据（包括队列名字属性、交换机的类型名字属性、绑定、vhost）放 在磁盘中。未指定类型的情况下，默认为磁盘节点。