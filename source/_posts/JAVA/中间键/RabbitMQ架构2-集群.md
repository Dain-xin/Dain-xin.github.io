---
title: RabbitMQ架构-集群
date: 2022-07-22 20:30:37
categories:
- 中间键
- 消息中间键
tag:
top:
keywords:
description:
cover: 'https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207222044058.png'
top_img: 'https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207222044058.png'
---

因为 guest 用户只能在本机访问，添加一个 admin 用户，密码也是 admin

```shell
 ./rabbitmqctl add_user admin admin 
 ./rabbitmqctl set_user_tags admin administrator 
 ./rabbitmqctl set_permissions -p / admin ".*" ".*" ".*
```

### 添加三个mq

```
docker pull rabbitmq:management

//单体部署
docker run -di --name=dain_rabbitmq -p 5672:5672 -p 5671:5671 -p 4369:4369 -p 15671:15671 -p 15672:15672 -p 25672:25672 rabbitmq:management

//集群部署 准备三个mq
docker run -d --hostname localhost --name rabbitmqCluster01 -p 15601:15672 -p 5601:5672 rabbitmq:management
docker run -d --hostname localhost --name rabbitmqCluster02 -p 15602:15672 -p 5602:5672 rabbitmq:management
docker run -d --hostname localhost --name rabbitmqCluster03 -p 15603:15672 -p 5603:5672 rabbitmq:management
```

### 容器节点加入集群

**首先**在centos窗口中，执行如下命令，进入第一个rabbitmq节点容器：

```javascript
docker exec -it rabbitmqCluster01 bash
```

进入容器后，操作rabbitmq,执行如下命令：

```javascript
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl start_app
exit
```

操作日志信息如下：

```javascript
[root@localhost rabbitmq01]# docker exec -it rabbitmqCluster01 bash
root@rabbitmq01:/# rabbitmqctl stop_app
Stopping rabbit application on node rabbit@rabbitmq01 ...
root@rabbitmq01:/# rabbitmqctl reset
Resetting node rabbit@rabbitmq01 ...
root@rabbitmq01:/# rabbitmqctl start_app
Starting node rabbit@rabbitmq01 ...
 completed with 3 plugins.
root@rabbitmq01:/# exit
exit
```

**接下来**，进入第二个rabbitmq节点容器，执行如下命令：

```javascript
docker exec -it rabbitmqCluster02 bash
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl join_cluster --ram rabbit@rabbitmq01
rabbitmqctl start_app
exit
```

操作日志信息如下：

```javascript
[root@localhost rabbitmq01]# docker exec -it rabbitmqCluster02 bash
root@rabbitmq02:/# rabbitmqctl stop_app
Stopping rabbit application on node rabbit@rabbitmq02 ...
root@rabbitmq02:/# rabbitmqctl reset
Resetting node rabbit@rabbitmq02 ...
root@rabbitmq02:/# rabbitmqctl join_cluster --ram rabbit@rabbitmq01
Clustering node rabbit@rabbitmq02 with rabbit@rabbitmq01
root@rabbitmq02:/# rabbitmqctl start_app
Starting node rabbit@rabbitmq02 ...
 completed with 3 plugins.
root@rabbitmq02:/# exit
exit
```

**最后**，进入第三个rabbitmq节点容器，执行如下命令：

```javascript
docker exec -it rabbitmqCluster03 bash
rabbitmqctl stop_app
rabbitmqctl reset
rabbitmqctl join_cluster --ram rabbit@rabbitmq01
rabbitmqctl start_app
exit
```

操作日志信息如下：

```javascript
[root@localhost rabbitmq01]# docker exec -it rabbitmqCluster03 bash
root@rabbitmq03:/#  rabbitmqctl stop_app
Stopping rabbit application on node rabbit@rabbitmq03 ...
root@rabbitmq03:/# rabbitmqctl reset
Resetting node rabbit@rabbitmq03 ...
root@rabbitmq03:/# rabbitmqctl join_cluster --ram rabbit@rabbitmq01
Clustering node rabbit@rabbitmq03 with rabbit@rabbitmq01
root@rabbitmq03:/# rabbitmqctl start_app
Starting node rabbit@rabbitmq03 ...
 completed with 3 plugins.
root@rabbitmq03:/# exit
exit
```
