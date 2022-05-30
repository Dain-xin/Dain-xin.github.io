---
title: 阿里云服务部署使用说明
date: 2022-05-29 15:47:17
categories:
- 环境搭建
- ECS
---

# 阿里云ECS云服务部署过程 

---

# 前言

如果想要自己的web项目可以通过公网访问，方法有很多这里采用阿里云的ECS来部署自己的第一个web项目

---

`提示：学生可以有优惠活动`

# 一、购买阿里云ECS
1、[飞天加速计划](https://developer.aliyun.com/plan/student#J_6054107750)

2、完成购买后进入控制台：[云服务器 ECS控制台](https://ecs.console.aliyun.com/#/home)
	

 - 找到实例与镜像在更多里面密码更改重启服务器。![重置密码](https://img-blog.csdnimg.cn/d91216a743684b328eda335e3e9ba56e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARGFpbi1Y,size_20,color_FFFFFF,t_70,g_se,x_16)
 - 使用windows连接工具（xshell）我的分享地址
 - 找到服务器的公网输入密码连接
![在这里插入图片描述](https://img-blog.csdnimg.cn/6e9f78285bb64b488a2a9eb1c0205dae.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARGFpbi1Y,size_20,color_FFFFFF,t_70,g_se,x_16)


# 二、安装Linux相关服务
安装内容：
下载分享：

## 1.JDK8
[阿里云JDK配置](https://developer.aliyun.com/article/773930)
>主要的代码：

```JAVA
export JAVA_HOME=/usr/local/java/jdk1.8.0_281   #你的JDK安装路径+名称
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export PATH=${JAVA_HOME}/bin:$PATH
```

## 2.Mysql 57
[Mysql57安装](https://developer.aliyun.com/article/834505)

1、如果出GPG的错：[GPG Keys are configured](https://cloud.tencent.com/developer/article/1940459)
2、出现“（”，“）”：[xshell字符转义](https://blog.csdn.net/wangchengming1/article/details/76946412)
3、mysql服务：密码过于简单需要设置：[地址](https://www.cnblogs.com/baby123/p/12221405.html)
4、navicat连接mysql出现错误 is not allowed to connect to this mysql server 的解决办法[链接](https://blog.csdn.net/weixin_42132143/article/details/118339697)

```bash
systemctl start mysqld.service
systemctl status mysqld.service
```
## 3.redis5
参考1：[阿里云Redis安装](https://developer.aliyun.com/article/775160)
参考2：[配置亲测有效](https://blog.csdn.net/lilong329329/article/details/90451657)
`重点：远程连接阿里云服务需要注释掉bind 127.0.0.1和设置密码，重启服务和配置文件`
![配置](https://img-blog.csdnimg.cn/5771d9a252aa4e15ac13eda283e1cad2.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBARGFpbi1Y,size_20,color_FFFFFF,t_70,g_se,x_16)
## 3.Tomcat 9
这个比较简单，把自己gz文件放到自己想要的位置 /usr/local/tomcat 中
tar -zxvf 文件名  解压好后，去往bin目录启动starup.sh。配置安全组8080，连接ip+8080即可。
## 4.Node.js
参考连接:[Node.js安装](https://developer.aliyun.com/article/775037)
## 5.Nacos
参考连接:[nacos安装](https://developer.aliyun.com/article/786617)
## 6.宝塔面板
参考连接:[面板配置](https://developer.aliyun.com/article/767507)
[参考2](https://developer.aliyun.com/article/766773#:~:text=%E5%AE%9D%E5%A1%94%E9%9D%A2%E6%9D%BF%E5%AE%89%E8%A3%85%E5%BE%88%E7%AE%80%E5%8D%95%EF%BC%8C%E4%B8%80%E6%9D%A1%E5%91%BD%E4%BB%A4%E5%8D%B3%E5%8F%AF%EF%BC%8C%E6%9C%AC%E6%95%99%E7%A8%8B%E4%BB%A5CentOS%E4%B8%BA%E4%BE%8B%EF%BC%8C%E5%AE%89%E8%A3%85%E5%91%BD%E4%BB%A4%E4%B8%BA%EF%BC%9A%20yum%20install%20-y%20wget%20&&,wget%20-O%20install.sh%20http://download.bt.cn/install/install.sh%20&&%20sh%20install.sh)
## 7.导出镜像
[导出自定义镜像](https://help.aliyun.com/document_detail/58181.html)

## 8.RabbitMq
[参考1](https://blog.csdn.net/qq_41854180/article/details/113832989)
[rpm安装过程](https://blog.csdn.net/u014528861/article/details/114370948)
[参考2](https://blog.csdn.net/feinifi/article/details/107997555)
## 9.Docker
[Docker使用](https://help.aliyun.com/document_detail/187598.html#section-kjt-i59-n4a)

# 总结
`提示：未完待续.......`
