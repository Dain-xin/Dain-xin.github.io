---
title: 增删改查BaseDao
date: 2022-07-05 08:54:51
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



# JDBC的初代封装原理

![image-20220705104012279](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207051040599.png)

## [利用反射技术将查询结果封装为对象](https://blog.csdn.net/buildingjiang/article/details/65443155)

![image-20220705105835364](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207051058653.png)

# JDBC封装优化1（动态代理+ResultSetMetaData）

 利用了动态代理和ResultSetMetaData类来封装了基础BaseDao类

1、动态代理：优化查询方法，自动调用对应的属性set方法完成对对象的封装；

2、ResultSetMetaData类：不需要知道属性名了，只需要获取对应属性名；在得到属性名对应的属性值，就可以得到对应的数据。

![image-20220705110331058](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207051103632.png)

# JDBC封装优化2

优化内容：

1、BaseDao的查询方法不需要在传参数 xxx.class 不需要在传，利用反射得到继承的父类的对象；

2、利用抽象方法，优化调用者不需要在重复写增删改查方法，直接实现对应的抽象方法即可。

![image-20220705110523879](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207051105008.png)

# 最终版BaseDao

![image-20220705110632628](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207051106832.png)