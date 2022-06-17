---
title: TreeSet和TreeMap理解
date: 2022-06-17 14:07:37
categories:
- JAVA
- 高级
- 集合
---

[Java集合类TreeMap和TreeSet](https://blog.csdn.net/speedme/article/details/22661671)

 

TreeMap 和 TreeSet 是 Java Collection Framework 的两个重要成员，其中 TreeMap 是 Map 接口的常用实现类，而 TreeSet 是 Set 接口的常用实现类。虽然 TreeMap 和TreeSet 实现的接口规范不同，但 **TreeSet** **底层是通过** **TreeMap** **来实现的（如同**HashSet底层是是通过HashMap来实现的一样），因此二者的实现方式完全一样。而 TreeMap 的实现就是红黑树算法

 

**TreeSet和TreeMap的关系**

与HashSet完全类似，TreeSet里面绝大部分方法都市直接调用TreeMap方法来实现的。

 

**相同点：**

- TreeMap和TreeSet都是非同步集合，因此他们不能在多线程之间共享，不过可以使用方法**Collections.synchroinzedMap()**来实现同步
- 运行速度都要比Hash集合慢，他们内部对元素的操作时间复杂度为O(logN)，而HashMap/HashSet则为O(1)。
- TreeMap和TreeSet都是有序的集合，也就是说他们存储的值都是拍好序的。

**不同点：**

- 最主要的区别就是TreeSet和TreeMap分别实现Set和Map接口
- TreeSet只存储一个对象，而TreeMap存储两个对象Key和Value（仅仅key对象有序）
- TreeSet中不能有重复对象，而TreeMap中可以存在
- **TreeMap**的底层采用红黑树的实现，完成数据有序的插入，排序。

