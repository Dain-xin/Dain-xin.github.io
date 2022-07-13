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





# 消息分发机制

就是Exchange的主要功能，RabbitMQ 中一共有四种类型的交换机，Direct、Topic、Fanout、Headers。

## Direct 直连



