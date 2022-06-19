---
title: 建库建表-DDL-DML
date: 2022-06-19 08:58:51
categories:
- 数据库
- mysql
---

# DDL操作数据库

![image-20220619091940787](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206190919956.png)

# DML操作表语句

![image-20220619092056387](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206190920457.png)

理解：

insert 的注意事项：

\1) 插入的数据应与字段的数据类型相同

\2) 数据的大小应在列的规定范围内，例如：不能将一个长度为 80 的字符串加入到长度为 40 的列中。

\3) 在 values 中列出的数据位置必须与被加入的列的排列位置相对应。在 mysql 中可以使用 value，但不建议使用，功能与 values 相同。

\4) 字符和日期型数据应包含在单引号中。MySQL 中也可以使用双引号做为分隔符。

\5) 不指定列或使用 null，表示插入空值。



# 设置全局编码

- 查看包含 character 开头的全局变量

 **show variables like 'character%';**

- 修改     client、connection、results 的编码为     GBK，保证和 DOS 命令行编码保持一致

 

| **单独设置**                          | **说明**                   |
| ------------------------------------- | -------------------------- |
| **set character_set_client=gbk;**     | 修改客户端的字符集为 GBK   |
| **set character_set_connection=gbk;** | 修改连接的字符集为 GBK     |
| **set character_set_results=gbk;**    | 修改查询的结果字符集为 GBK |

- 同时设置三项

 **set names gbk;**

 

# **蠕虫复制：**

##  **什么是蠕虫复制：**

将一张已经存在的表中的数据复制到另一张表中。

 

## **语法格式：**

```mysql
-- 将表名 2 中的所有的列复制到表名 1 中

INSERT INTO 表 名 1 SELECT * FROM 表 名 2;  

-- 只复制部分列

INSERT INTO 表 名 1( 列 1, 列 2) SELECT 列 1, 列 2 FROM student;
```

