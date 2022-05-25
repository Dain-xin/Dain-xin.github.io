---
title: Spring Cloud学习笔记1
date: 2022-05-25 22:50:17
categories:
- JAVA
- Spring Cloud
---

网关组测到nacos，访问getway网关，根据路由规则访问注册再nacos中的服务，根据ribbon的负载均衡访问一个服务名的多个服务。

（网关访问，可以进行限流、降级、熔断等操作——容错机制——断路器）

熔断器的作用：在微服务中，如果上层服务没有作好熔断机制，当一个请求访问服务，未得到处理，就会一直占用资源，不会释放，这样就会导致服务被很多服务请求拖垮，就会导致服务雪崩（级联效应）。

处理方式：

1. 超时：如果服务连接超时、即使未成功也会被释放资源；
2. 仓壁模式：就是线程池，
3. 限流功能-Sentinel，
4. 断路器模式，通过下图，让我们的请求处于closed状态，如果我们的服务调用失败率到达一定的值域就会open状态，然后进行重置超时，成功就closed，否则就继续open；

![image-20220525203610863](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202205252036007.png)

Sentinel：限流

例如：我们启动Sentinel服务，监控8080端口，（三板斧，pom导入jar、yml指定Sentinel-dashboard）把我们的服务发送给我们的Sentinel控制台但是不显示，懒加载，访问后可见服务分级，通过Sentinel控制台设置限流规则（就是我们再Sentinel控制台服务端设置规则，发送给我们的客户端，然后就可以生效我们的限流规则）

Sentinel的持久化规则：

1. pull模式
2. push模式，通过我们watch监听器

Sentinel云原生

![image-20220525210640723](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202205252106881.png)

链路追踪：我们进行一次服务调用，但是无法发现我们的服务调用的路径（网关、服务提供、服务消费、其他微服务）可以排查我们的服务出错的位置。

zipkin+Selush、skywalking

9：30