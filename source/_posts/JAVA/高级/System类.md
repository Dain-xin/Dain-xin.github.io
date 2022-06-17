---
title: System类
date: 2022-06-17 09:50:37
categories:
- JAVA
- 高级
---

System类：（lang包）

[Java System类详解](http://c.biancheng.net/view/904.html)

1、类常用的方法

|               static  long currentTimeMillis()               |                 返回以毫秒为单位的当前时间。                 |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| static  void arraycopy(Object  src, int srcPos,   Object  dest, int destPos, int length) |            将数组中指定的数据拷贝到另一个数组中。            |
|                      static  void gc()                       | 调用 gc  方法暗示着 Java 虚拟机做了一些**努力来回收未用对象**，以便能够快速地重用这些对象当前占用的内存。（但是不一定能回收到对象，没有强制性，从所有丢弃的对象中回收了空间） |
|                   static exit (int status)                   | 终止当前正在运行的 Java 虚拟机;  status 的值为 0 时表示正常退出，非零时表示异常退出。使用该方法可以在图形界面编程中实现程序的退出功能等 |

 

arraycopy参数理解

| 1    | src  Object  | 源数组               |
| ---- | ------------ | -------------------- |
| 2    | srcPos  int  | 源数组索引起始位置   |
| 3    | dest Object  | 目标数组             |
| 4    | destPos  int | 目标数组索引起始位置 |
| 5    | length int   | 复制元素个数         |

**arraycopy(dataType[] srcArray,int srcIndex,int destArray,int destIndex,int length) 方法**

```java
System.arraycopy(src,0,temp,2,2);
System.arraycopy(src,4,temp,4,2);

//理解
//结果：null null （X X）（X X） null null 
//复制src第一个字符开始两个字符，给temp数组，存在temp的第三个字符的位置；
//复制src第五个字符开始两个，给temp数组，在temp的第五个字符的位置）

*/**
*src-**源数组。*
*srcPos-**源数组中的起始位置。*
*dest-**目标数组。*
*destPos-**目标数据中的起始位置。*
*length-**要复制的数组元素的数量。*
**/*
*//**这两具数组都有自己独立的堆内存空间
```

![image-20220617103056020](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206171030566.png)