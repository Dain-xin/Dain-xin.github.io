---
title: Collections工具类
date: 2022-06-17 14:40:37
categories:
- JAVA
- 高级
- 集合
---

[Java Collections类操作集合详解 ](http://c.biancheng.net/view/6884.html#:~:text=Collections 类是 Java 提供的一个操作 Set、List 和 Map 等集合的工具类,类提供了许多操作集合的静态方法，借助这些静态方法可以实现集合元素的排序、查找替换和复制等操作。 下面介绍 Collections 类中操作集合的常用方法。 Collections 提供了如下方法用于对 List 集合元素进行排序。) 

1、（Util包）Collections 类是 Java 提供的一个操作 Set、List 和 Map 等集合的工具类。Collections 类提供了许多操作集合的**静态**方法**，借助这些静态方法可以实现集合元素的**排序、查找替换和复制等操作。

 

2、常见排序方法正需：

| void reverse(List list)：               | 对指定 List 集合元素进行逆向排序。                           |
| :-------------------------------------- | ------------------------------------------------------------ |
| void shuffle(List list)：               | 对 List 集合元素进行随机排序（shuffle 方法模拟了“洗牌”动作）。 |
| void sort(List list)：                  | 根据元素的自然顺序对指定 List 集合的元素按升序进行排序。     |
| void sort(List list, Comparator  c)     | 根据指定 Comparator 产生的顺序对 List 集合元素进行排序。     |
| void swap(List list, int i, int  j)：   | 将指定 List 集合中的 i 处元素和 j 处元素进行交换。           |
| void rotate(List list, int  distance)： | 当 distance 为正数时，将 list 集合的后  distance 个元素“整体”移到前面；当 distance 为负数时，将 list 集合的前 distance  个元素“整体”移到后面。该方法不会改变集合的长度。 |

 

eg：

```java
// 调用reverse()方法对集合元素进行反转排序

Collections.reverse(students); 

// 调用sort()方法对集合进行排序

Collections.sort(prices);
```

3、查找替换集合方法。

| int binarySearch(List list, Object key)：                    | 使用二分搜索法搜索指定的  List 集合，以获得指定对象在 List 集合中的索引。如果要使该方法可以正常工作，则必须保证 List 中的元素已经处于有序状态。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| Object max(Collection  coll)：                               | 根据元素的自然顺序，返回给定集合中的最大元素。               |
| Object max(Collection coll,  Comparator comp)：              | 根据 Comparator  指定的顺序，返回给定集合中的最大元素。      |
| Object **min**(Collection coll)：                            | 根据元素的自然顺序，返回给定集合中的最小元素。               |
| Object min(Collection coll,  Comparator comp)：              | 根据 Comparator  指定的顺序，返回给定集合中的最小元素。      |
| void fill(List list, Object  obj)：                          | 使用指定元素 obj 替换指定 List 集合中的所有元素。            |
| int frequency(Collection c,  Object o)：                     | 返回指定集合中指定元素的出现次数。                           |
| int indexOfSubList(List source,  List target)：              | 返回子 List 对象在父 List  对象中第一次出现的位置索引；如果父 List 中没有出现这样的子 List，则返回 -1。 |
| boolean replaceAll(List list, Object oldVal, Object newVal)： | 使用一个新值 newVal 替换 List 对象的所有旧值  oldVal。       |

 

eg：



```java
//max：返回集合中最大的数据

ArrayList<String>arrayList=newArrayList();

String max=Collections.max(arrayList);

//fill 替换所有元素；但是replaceAll替换旧数据成新的数据（较为实用）

ArrayList<String>arrayList=newArrayList();

arrayList.add("1");

arrayList.add("2");

arrayList.add("3");

arrayList.add("4");

ArrayList<String>arrayList2=newArrayList();

arrayList2.add("a");

arrayList2.add("b");

arrayList2.add("c");

arrayList2.add("d");

Stringmax=Collections.max(arrayList);

Collections.fill(arrayList2,"e");

Collections.replaceAll(arrayList,"4","5");

System.out.println("arrayList="+arrayList);

System.out.println("arrayList2="+arrayList2);
```

结果：

arrayList = [1, 2, 3, 5]

arrayList2 = [e, e, e, e]

4、复制方法：

a、Collections 类的 copy() 静态方法用于将指定集合中的所有元素复制到另一个集合中。执行 copy() 方法后，目标集合中每个已复制元素的索引将等同于源集合中该元素的索引。

b、void copy(List <? super T> dest,List<? extends T> src)

其中，dest 表示目标集合对象，src 表示源集合对象。

注意：目标集合的长度至少和源集合的长度相同，如果目标集合的长度更长，则不影响目标集合中的其余元素。如果目标集合长度不够而无法包含整个源集合元素，程序将抛出 IndexOutOfBoundsException 异常。

d、eg： 调用copy()方法将当前商品信息复制到原有商品信息集合中

Collections.copy(destList, srcList);

 

 

 