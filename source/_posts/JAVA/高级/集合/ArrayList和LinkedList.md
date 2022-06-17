---
title: ArrayList和LinkedList
date: 2022-06-03 21:22:14
categories:
- JAVA
- 高级
- 集合
---

# List

1、java.util.List 接口继承自 Collection 接口，是单列集合的一个重要分支，习惯性地会将实现了 List 接口的对象称为List集合。

2、在List集合中允许出现重复的元素，所有的元素是以一种线性方式进行存储的，在程序中可以通过 索引来访问集合中的指定元素

3、List集合还有一个特点就是元素有序，即元素的存入顺序和取出顺序一致。

4、常用方法：

List接口常用方法：

| add(Object  element)：               | 向列表的尾部添加指定的元素。                                 |
| ------------------------------------ | ------------------------------------------------------------ |
| size()：                             | 返回列表中的元素个数。                                       |
| **get(int index)**：                 | 返回列表中指定位置的元素，index从0开始。                     |
| **add(int index, Object element)**： | 在列表的指定位置插入指定元素。                               |
| **set(int i, Object element)**：     | 将索引i位置元素替换为元素element并返回被替换的元素。         |
| clear()：                            | 从列表中移除所有元素。                                       |
| isEmpty()：                          | 判断列表是否包含元素，不包含元素则返回  true，否则返回false。 |
| contains(Object  o)：                | 如果列表包含指定的元素，则返回  true。                       |
| **remove(int index)**：              | 移除列表中指定位置的元素，并返回被删元素。                   |
| **remove(Object o)**：               | 移除集合中第一次出现的指定元素，移除成功返回true，否则返回false。 |
| **iterator()**：                     | 返回按适当顺序在列表的元素上进行迭代的迭代器。               |

 

注：**ArrayList有List的所有方法，LinkedLIst也具有这些方法。**

 



# LinkedList 特殊方法

| public  void addFirst(E e) : | 将指定元素插入此列表的开头。         |
| ---------------------------- | ------------------------------------ |
| public  void addLast(E e) :  | 将指定元素添加到此列表的结尾。       |
| public E  getFirst() :       | 返回此列表的第一个元素。             |
| public E  getLast() :        | 返回此列表的最后一个元素。           |
| public E  removeFirst() :    | 移除并返回此列表的第一个元素。       |
| public E  removeLast() :     | 移除并返回此列表的最后一个元素。     |
| public E pop() :             | 从此列表所表示的堆栈处弹出一个元素。 |
| public void push(E e) :      | 将元素推入此列表所表示的堆栈。       |
| public  boolean isEmpty() ： | 如果列表不包含元素，则返回true。     |

注：**这些方法结合使用可以实现栈、和队列的效果。**

 

# ArrayList  vs. LinkedList。

## 相同

LinkedeList和ArrayList是常用的两种存储结构，都可以实现了List接口，两者结构都线程不安全，

## 不同

1、数据结构不同

ArrayList是Array(动态数组)的数据结构，LinkedList是Link(链表)的数据结构。

2、效率不同

当随机访问List（get和set操作）时，ArrayList比LinkedList的效率更高，因为LinkedList是线性的数据存储方式，所以需要移动指针从前往后依次查找。

当对数据进行增加和删除的操作(add和remove操作)时，LinkedList比ArrayList的效率更高，因为ArrayList是数组，所以在其中进行增删操作时，会对操作点之后所有数据的下标索引造成影响，需要进行数据的移动。

3、自由性不同

ArrayList自由性较低，因为它需要手动的设置固定大小的容量，但是它的使用比较方便，只需要创建，然后添加数据，通过调用下标进行使用；而LinkedList自由性较高，能够动态的随数据量的变化而变化，但是它不便于使用。

4、主要控件开销不同

ArrayList主要控件开销在于需要在lList列表预留一定空间；而LinkList主要控件开销在于需要存储结点信息以及结点指针信息。

例如：

 ArrayList 的每个索引的位置是实际的数据，

LinkedList 需要更多的内存， LinkedList 中的每个节点中存储的是实际的数据和前后节点的位置 ( 一个 LinkedList 实例存储了两个值： Node<E> first 和 Node<E> last 分别表示**链表的其实节点和尾节点**，每个 Node 实例存储了三个值： E item，Node next，Node pre) 。

**理解：**

**主要还是数据结构的问题，array数组的，刚开始长度一定，需要预先分配好，下好资源，但是查询快的特点；**

**另一个是link链表的数据结构，由结点组成；但是查询比较麻烦，但是插入数据简单只需指明结点位置。**

# linkedList和ArrayList适宜的场景

1) 你的应用不会随机访问数据 。因为如果你需要LinkedList中的第n个元素的时候，你需要从第一个元素顺序数到第n个数据，然后读取数据。

2) 你的应用更多的插入和删除元素，更少的读取数据 。因为插入和删除元素不涉及重排数据，所以它要比ArrayList要快。

> 换句话说，ArrayList的实现用的是数组，LinkedList是基于链表，ArrayList适合查找，LinkedList适合增删
