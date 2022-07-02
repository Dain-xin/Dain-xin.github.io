---
title: Centos项目环境搭建
date: 2022-06-30 15:47:17
categories:
- 环境搭建
- centos
tag:
top:
keywords:
description:
cover: 'https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206301138862.png'
top_img: 'https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206301138862.png'
---

# 安装宝塔面板

```sh
yum install -y wget && wget -O install.sh http://download.bt.cn/install/install_6.0.sh && sh install.sh ed8484bec   
//宝塔centos安装命令
```

```text
外网面板地址: http://117.155.26.175:40653/489457aa
内网面板地址: http://192.168.100.140:40653/489457aa
*以下仅为初始默认账户密码，若无法登录请执行bt命令重置账户/密码登录
username: r9h0jeos
password: 0809d4fc
If you cannot access the panel,
release the following panel port [40653] in the security group
若无法访问面板，请检查防火墙/安全组是否有放行面板[40653]端口
```



## 关闭防火墙

```sh
systemctl stop firewalld  //关闭防火墙
```

## 内网登录

下载Docker

![image-20220630113815258](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206301138862.png)



## 宝塔安装Nginx、Node管理、Mysql、Docker、Php、SSH

在软件商店下载即可，注意版本需要我选择的

![image-20220630140152801](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206301401843.png)



# Docker安装

## Docker安装Nacos

http://192.168.100.140:8848/nacos/

![image-20220630140533585](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206301405643.png)

```shell
docker pull nacos/nacos-server

docker images

docker run --env MODE=standalone --name dain_nacos -d -p 8848:8848 docker.io/nacos/nacos-server

内存不足使用
docker run -e JVM_XMS=128m -e JVM_XMX=128m --env MODE=standalone --name dain_nacos -d -p 8848:8848 docker.io/nacos/nacos-server

```

## Docker安装Redis

```
docker pull redis:latest
docker run -itd --name dain-redis -p 6379:6379 redis
```

![image-20220630142256269](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206301422329.png)

## Docker安装RabbitMQ

[http://192.168.100.140:15672](http://192.168.100.140:15672/)

```
docker pull rabbitmq:management

docker run -di --name=dain_rabbitmq -p 5672:5672 -p 5671:5671 -p 4369:4369 -p 15671:15671 -p 15672:15672 -p 25672:25672 rabbitmq:management
```

无需新建用户直接可以使用guest：guest登录

![image-20220630140048466](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206301400540.png)

![image-20220630140126087](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206301401136.png)



# Docker安装结果

![image-20220630142352787](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206301423834.png)

![image-20220630142406784](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206301424830.png)
