---
title: HashSet和LinkHashMap理解
date: 2022-06-17 13:03:37
categories:
- JAVA
- 高级
- 集合
---

# set接口

1、java.util.HashSet 是 Set 接口的一个实现类，它所存储的**元素是不可重复的，并且元素都是无序的(即存取顺序 不一致)**。 

2**、java.util.HashSet 底层的实现其实是一个 java.util.HashMap 支持**，HashSet 是根据对象的哈希值来确定元素在集合中的存储位置，因此具有良好的存取和查找性能。保证元素唯一性 的方式依赖于： hashCode 与 equals 方法。 

3、HashSet保证元素唯一，可是元素存放进去是没有顺序的，那么我们要**保证有序**，

HashSet下面有一个子类 java.**util**.LinkedHashSet ，它是**链表和哈希表组合的一个数据存储结构**。

 

注：LinkedHashSet的底层就是LinkedHashMap来实现。

4、HashSet存储对象HashCode值相同的处理。

![06_Set集合存储元素不重复的原理](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206171332344.bmp)



# [**HashMap和HashSet的区别**](https://blog.csdn.net/chen213wb/article/details/84647179?utm_medium=distribute.pc_relevant_t0.none-task-blog-2~default~BlogCommendFromBaidu~default-1.control&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-2~default~BlogCommendFromBaidu~default-1.control)**：**

 

|                  *HashMap*                  |                          *HashSet*                           |
| :-----------------------------------------: | :----------------------------------------------------------: |
|            HashMap实现了Map接口             |                     HashSet实现了Set接口                     |
|              HashMap储存键值对              |                     HashSet仅仅存储对象                      |
|         使用put(方法将元素放入map中         |                 使用add0方法将元素放入set中                  |
|     HashMap中使用键对象来计算hashcode值     | HashSet使用成员对象来计算hashcode值，对于两个对象来说hashcode可能相同，所以equals(方法用来判断对象的相等性，如果两个对象不同的话，那么返回false |
| HashMap比较快，因为是使用唯一的键来获取对象 |                  HashSet较HashMap来说比较慢                  |

 

**注：HashSet**底层时用HashMap来实现；LinkedHashSet底层时LinkedHashSMap实现。

 

## **HashMap**：源码级别理解：

**1**、结构上是一个扩充的数组，需要一下属性；

- **capacity**：目前数组的长度。为了实现高效的扩容，其值总为2^n的形式。每次扩容后，n会加1，即整个数组的容量变为之前的    2倍。该值初始默认值为16。
- **loadFactor**：负载因子，默认值为     0.75。该值与threshold配合使用。
- **threshold**：扩容的阈值，等于 **capacity     \* loadFactor**。即当数组内达到这么多元素时，会**触发数组的扩容**。

**2**、数组中的每个元素是一个Node，它的属性有：

- hash: 当前位置值的hash值
- key：当前位置的键
- value: 当前位置存储的值
- next;下一个Node

3、JDK7，Node的存储还是单链表；JDK8改用红黑树提高效率**（final void** **treeifyBin**(Node<K,V>[] tab, int hash)

4、数据写入（重点）

向HashMap中写入数据的过程，简单总结起来分为这么几步：

- 计算要插入数据的Hash值，并根据该值确定元素的插入位置（即在动态数组中的位置）。
- 将元素放入到数组的指定位置



- 如果该数组位置之前没有元素，则直接放入
- 放入该位置后，数组元素超过扩容阈值，则对数组进行扩容
- 放入该位置后，数组元素没超过扩容阈值，写入结束



- 如果该数组位置之前有元素，则挂载到已有元素的后端
- 如果之前元素组成了树，则挂入树的指定位置
- 如果之前元素组成了链表
- 如果加入该元素链表长度超过8，则将链表转化为红黑树后插入
- 如果加入该元素链表长度不超过8，则直接插入

 

 

# **HashMap和HashSet存储对象过程：**

 

**一、HahMap存储对象的过程如下**

1、对HahMap的Key调用hashCode()方法，返回int值，即对应的hashCode；

2、把此hashCode作为哈希表的索引，查找哈希表的相应位置，若当前位置内容为NULL，则把hashMap的Key、Value包装成Entry数组，放入当前位置；

3、若当前位置内容不为空，则继续查找当前索引处存放的链表，利用equals方法，找到Key相同的Entry数组，则用当前Value去替换旧的Value；

4、若未找到与当前Key值相同的对象，则把当前位置的链表后移（Entry数组持有一个指向下一个元素的引用），把新的Entry数组放到链表表头；

 

**二、HashSet存储对象的过程**

**1**、java.util.HashSet 底层的实现其实是一个 java.util.HashMap 支持；

**2**、add、方法就是、Map、的put方法（key，(静态Object) **）；构造方法**new HashMap().

 

**源码：**

```java
private static final Object  PRESENT = new Object();

//**1**、构造方法。

public HashSet(){

map=new HashMap<>();

}

//**2**、add方法。

public boolean add(E e){

return map.put(e,PRESENT)==null;

}

//**3**、remove方法。

public boolean remove(Object o){

return map.remove(o)==PRESENT;  //remove根据key删除后返回对应的值

}
```

 

3、往HashSet添加元素的时候，HashSet会先调用元素的**hashCode方法**得到元素的哈希值 ，然后通过元素 的哈希值经过移位等运算，就可以算出该元素在哈希表中 的存储位置。

 

### **存储图解**：

**HashSet集合存储数据的结构(哈希表)**

**jdk1.8版本之前:哈希表=数组+链表**

**jdk1.8版本之后:**

**哈希表=数组+链表;**

**哈希表=数组+红黑树(提高查询的速度)哈希表的特点:速度快**

![image-20220617134601754](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206171346810.png)

**情况1： 如果算出元素存储的位置目前没有任何元素存储，那么该元素可以直接存储到该位置上。**

**情况2： 如果算出该元素的存储位置目前已经存在有其他的元素了，那么会调用该元素的equals方法与该位置的元素再比较一次，如果equals返回的是true，那么该元素与这个位置上的元素就视为重复元素，不允许添加，如果equals方法返回的是false，那么该元素运行添加。**



# 存储对象的理解

**JDK1.8引入红黑树大程度优化了HashMap的性能，那么对于我们来讲保证HashSet集合元素的唯一， 其实就是根据对象的hashCode和equals方法来决定的。**如果我们往集合中存放自定义的对象，那么保证其唯一， 就必须复写hashCode和equals方法建立属于当前对象的比较方式。



# 树参考图

![微信图片_20220617134846](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206171349399.png)