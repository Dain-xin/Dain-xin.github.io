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

Erlang 是为电话 交换机编写的语言，天生适合分布式和高并发。

国内的绝大部分大厂都在用 RabbitMQ，包括头条，美团，滴滴（TMD），去哪儿， 艺龙，淘宝也有用。 

RabbitMQ 和 Spring 家族属于同一家公司：Pivotal。 

除了 AMQP 之外，RabbitMQ 支持多种协议，STOMP、MQTT、HTTP、 WebSockets

# RabbitMQ架构原理

RabbitMQ 实现了 AMQP 协议，所以 RabbitMQ 的工作模型也是基于 AMQP 的。

![image-20220711093357648](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207110933720.png)