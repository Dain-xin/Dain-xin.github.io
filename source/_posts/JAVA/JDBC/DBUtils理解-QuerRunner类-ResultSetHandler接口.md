---
title: DBUtils理解-QuerRunner类-ResultSetHandler接口
date: 2022-07-05 14:54:51
categories:
- JAVA
- JDBC
tag:
top:
keywords:
description:
cover: 'https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207051058653.png'
top_img: 'https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207051058653.png'
---

[什么是Java内存泄漏 - 简书 (jianshu.com)](https://www.jianshu.com/p/f847a383eec0)

 

在Java中，这些不可达的对象都由GC负责回收，因此程序员不需要考虑这部分的内存泄露。

# **内存泄漏案例：**

1、static大量使用；

2、资源未关闭（Connection，ResultSet等）；

3、不正确使用equals和hashcode方法；

4、使用外部类的内部类；

5、finalize（）：使对象不会立马被GC回收；

6、ThreadLocal（使用ThreadLocal时，每个线程只要处于存活状态就可保留对其ThreadLocal变量副本的隐式调用，

且将保留其自己的副本。如果线程不存在，该线程ThreadLocal会被GC回收但是保存的变量副本不会，就会导致内存泄漏）。

 

# **为什莫用连接池：**

常用的数据库连接使用步骤，会造成大量的资源浪费，频繁的连接断开可能会导致数据库内存泄漏，引起重启数据库；同时连接大量数，可能会导致内存泄漏，引起数据库崩溃。

 

# **连接池技术优点：**

\1. 资源重用。由于数据库连接得以重用，避免了频繁创建，释放连接引起的大量性能开销。

\2. 更快的系统反应速度。避免了数据库连接初始化和 释放过程的时间开销，从而减少了系统的响应时间 

\3. 新的资源分配手段。避免某一应用独占所有的数据库资源 

\4. 统一的连接管理，避免数据库连接泄漏 在较为完善的数据库连接池实现中，可根据预先的占用超时设定，强制回收被占用连接，从而避免 了常规数据库连接操作中可能出现的资源泄露

 

# **JDBC** **的数据库连接池**

**使用** **javax.sql.DataSource** **来表示：**

DataSource 只是一个接口，该接口通 常由服务器(Weblogic, WebSphere, Tomcat)提供实现；

DataSource 通常被称为数据源，它包含连接池和连接池管理两个部分，习惯上也经常把 DataSource 称为连接池 

DataSource用来取代DriverManager来获取Connection，获取速度快，同时可以大幅度提高数 据库访问速度。

 

## **开源组织：**

1、DBCP 是Apache提供的数据库连接池。tomcat 服务器自带dbcp数据库连接池。速度相对 c3p0较快，但因自身存在BUG，Hibernate3已不再提供支持。 

2、C3P0 是一个开源组织提供的一个数据库连接池，速度相对较慢，稳定性还可以。hibernate 官方推荐使用 

3、Proxool 是sourceforge下的一个开源项目数据库连接池，有监控连接池状态的功能，稳定性 较c3p0差一点 BoneCP 是一个开源组织提供的数据库连接池，速度快 

4、Druid 是阿里提供的数据库连接池，据说是集DBCP 、C3P0 、Proxool 优点于一身的数据库 连接池，但是速度不确定是否有BoneCP快 

## **DBUtils简介：**

**API：**

org.apache.commons.dbutils.QueryRunner 

org.apache.commons.dbutils.ResultSetHandler 

工具类：org.apache.commons.dbutils.DbUtils 

![image-20220705151622590](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207051516650.png)

## **QueryQunner类：**

**两个构造方法：一个默认的，另一个需要DataSource可以自己关闭该资源。**

**主要方法：增删改查、批处理****batch**

 

## **ResultSetHandler接口：**

只有一个单独的方法：

Object handle (java.sql.ResultSet .rs)。 

### **实现类：**

**ArrayHandler**：把结果集中的第一行数据转成对象数组。 

**ArrayListHandler**：把结果集中的每一行数据都转成一个数组，再存放到List中。 

**BeanHandler**：将结果集中的第一行数据封装到一个对应的JavaBean实例中。 

**BeanListHandler**：将结果集中的每一行数据都封装到一个对应的JavaBean实例中，存放到 List里。 ColumnListHandler：将结果集中某一列的数据存放到List中。 

**KeyedHandler**(name)：将结果集中的每一行数据都封装到一个Map里，再把这些map再存 到一个map里，其key为指定的key。 

**MapHandler**：将结果集中的第一行数据封装到一个Map里，key是列名，value就是对应的 值。 MapListHandler：将结果集中的每一行数据都封装到一个Map里，然后再存放到List 

**ScalarHandler**：查询单个值对象

 