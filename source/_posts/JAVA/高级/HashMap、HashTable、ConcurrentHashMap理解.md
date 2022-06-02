---
title: HashMap、HashTable、ConcurrentHashMap理解
date: 2022-06-02 22:20:29
categories:
- JAVA
- 高级
- 集合
tags:
- 面试
---

### hashMap和hashTable区别

1. HashMap是线程不安全的，在多线程环境下会容易产生死循环，但是单线程环境下运行效率高；Hashtable线程安全的，很多方法都有synchronized修饰，但同时因为加锁导致单线程环境下效率较低。
2. HashMap允许有一个key为null，允许多个value为null；而Hashtable不允许key或者value为null。
3. HashMap的底层数组的长度必须为2^n，这样做的好处是为以后的hash算法做准备，而Hashtable底层数组的长度可以为任意值，这就造成了当底层数组长度为合数的时候，Hashtable的hash算法散射不均匀，容易产生hash冲突。
4. 扩容都是基于底层的hash算法（hashMap的hash算法更优）；HashMap数组的扩容的整体思想就是创建一个长度为原先2倍的数组。然后对原数组进行遍历和复制。Hashtable的扩容将先创建一个长度为原长度2倍的数组，再使用头插法将链表进行反序。
5. HashMap在jdk1.8在原先的数组+链表的结构进行了优化，将实现结构变为数组+链表+红黑树；Hashtable到了jdk1.8了内部结构并没有实质优化，继续使用数组+链表的方式实现。
6. 可以使用**ConcurrentHashMap来替代Hashtable来使用**。

### [HashMap理解](https://zhuanlan.zhihu.com/p/513522232)

HashMap 1.7是由数组和链表实现的，初始长度是2^3，需要一个key-value的数据结构的集合，可以通过key访问value。因此产生了数组链表和散列机制。

![img](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206011426025.jpeg)

散列价值是速度，散列让我们查询更快。

#### **散列技术**

根据结点通过散列函数，得到的关键码值来确定其存储地址。关键码与散列地址是一对一的关系，则在检索时只需根据散列函数对给定值进行某种运算，即可得到待查结点的存储位置。但是，散列函数可能对于**不相等的关键码计算出相同的散列地址**，我们称该现象为**冲突**

数组链表, 便是定义一个数组, 然后数组的每一个成员都是一条链表,数组只需要记载这条链表的引用即可, 这样不需要直接在数组内部存储 键-值 对而需要大量的连续的内存空间。

#### **HashMap的延迟初始化和长度**

HashMap的构造法方法在执行时会初始化一个数组table，大小为0。

HashMap的put方法在执行时，首先会判断table的大小是否为0.如果为0，则会进行真正的初始化，也被称为延迟加载。

当进行真正初始化时，数组的默认大小为16。当然也可以调用HASHMAP的有参构造，自己DIY一个数组的初始化容量（但是每次都是2的幂的数组长度）

#### HashMap的数组下标如何产生

put的数据，把key和value存入，存入数组要有下标，就是来源于hash算法，HashMap通过hash算法把key变成数组下标。但是hash达到的数都是比较大的不可以直接作为下标，所以通过hashcode值变成二进制，和15二进制相与，得到符合下标的范围：

h&（length-1）则为：

h:   0101 0101

15: 0000 1111 

&-->0000 0101

对于上面这个运行结果的取值方法，我们来讨论一下：因为15的高四位都是0，低四位都是1。而与操作的逻辑是两个运算位都是1结果才是1。所以对于上面这个运算结果的高四位肯定都是0（15高四位都是0），而低四位和h的低四位是一样的（15低四位都是1）。所以结果的取值范围就是h的低四位的一个取值范围：0000~1111，也就是0至15（把高位全部丢弃，压缩至低位）。

这也是长度为什莫要是2的幂的原因。

#### HashMap1.7采用头插法

当put发生冲突。用链表解决，但是在1.7使用的是头插法插入，虽然有点反人类，但是如果插入的数据集较多，但是后插入的数据更有可能被查询。这也就是头插法的原因。

头插法实现过程：

![image-20220601131217549](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206011312700.png)

将节点3插入链表头部，当hash结果定位到节点所在的数组中，比较value值；再插入结束后链表向下移动一位。但是如果key相同，覆盖value，返回被覆盖额value。

![image-20220601131505093](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206011315169.png)

注意：头插法的目的是在获取元素时，新插入的元素被访问的频率更高。获取元素时的key一定是唯一的，因此访问到第一个符合的key值就不需要继续遍历了。

#### HashMap不适合并发

push方法非安全，如果在多线程下，线程1添加B，但是线程2添加的C刚好和B重复，但是由于连个相乘插入push数据是不可见的，就会发生覆盖造成不安全。

#### HashMap扩容问题

HashMap扩容的必要性：当我们插入的数据越来越多的时候，为了提升查询效率，扩容范围在0.75，每次扩容都是在2的幂指数。

但是扩容也会导致死循环，扩容的transfer方法中，外层for循环控制着数组遍历，内存的while循环控制着链表的遍历。

##### 单线程扩容过程：

先生成一个长度为oldTable数组长度2倍的newTable，然后将原本oldTable上的元素一次依照transfer方法的逻辑，添加至newTable中。

**1：执行e.next = newTable[i];**

![img](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206011355078.jpeg)

> 我们的e经过外层for循环定位到了数组下标为1的链表中。并且指向链表的第一个元素，也就是a。此时的newTable作为一个新生成的容器，并没有任何元素。因此e.next = newTable[i] = null;

**2：执行newTable[i]=e；**

![img](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206011400041.jpeg)

> 我们都知道，e指向的是a，因此newTable[i]=e，也就是把元素a挂在了newTable下标为i的链表中。

**3：执行e=next；**

![img](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206011400084.jpeg)

> 仔细观察代码，我们发现在while循环进入时，我们执行了Entry<K,V> next = e.next；也就是说，我们提前将节点b保存在了next字段中。此时执行e = next；相当于a的指针向后移动至b。结束第一轮循环。

**4：再次执行e.next = newTable[i];**

![img](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206011400126.jpeg)

> 此时的newTable[i]已经不再是null了，而是经过第一次循环变为了a。因此此步骤相当于e.next=a;

**5：再次执行newTable[i]=e；**

![img](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206011400155.jpeg)

> 将b用头插法插入newTable[i]中。

**6：再次执行e=next；**

![img](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/v2-d1d20dbb2cb3295ad2aa390eec282533_720w.jpg)

> 此时再次从元素b开始向后移动指针。发现后续元素已经为null。从而结束整个oldTable[1]的链表遍历。

**总结**

![img](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206011400284.jpeg)

> 整个操作仔细看下来，我们发现头插法的一个特性。就是在扩容过程中再次使用头插法，数据就会出现和原先HashMap链表逆序的现象

##### 多线程下扩容可能死循环

如果线程1让我们的链表倒序，其实也就是在单线程下，本来oldTable遍历完毕next就是null，但是如果多线程下线程2也执行了倒序操作，让我们e.next还有数据，就产生了死循环。

#### JDK1.8解决了死循环问题

1.7是用单链表进行的纵向延伸，当采用头插法时会容易出现逆序且环形链表死循环问题。

1.8之后是因为加入了红黑树使用尾插法，能够避免出现逆序且链表死循环的问题，也能更加高效的对链表进行遍历。

### [ConcurrentHashMap](https://zhuanlan.zhihu.com/p/50675786)理解

1. 并发容器，在concurrent并发包中，是对HashMap的安全实现，也是数组和链表的结构。

2. 但是在HashEntry和Segment都是静态内部类，基本和hashMap类似，但是ConcurrentHashMap中HashEntry的value和next指针都呗volatille修饰，保证了数据的可见性；

   ```
   static final class Segment<K,V> extends ReentrantLock implements Serializable {
          private static final long serialVersionUID = 2249069246763182397L;
          
          // 和 HashMap 中的 HashEntry 作用一样，真正存放数据的桶
          transient volatile HashEntry<K,V>[] table;
          transient int count;
          transient int modCount;
          transient int threshold;
          final float loadFactor;
          
   }
   ```

   ![image-20220602093324445](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206020933572.png)

3. ConcurrentHashMap 采用了分段锁技术，其中 Segment 继承于 ReentrantLock。不会像 HashTable 那样不管是 put 还是 get 操作都需要做同步处理，理论上 ConcurrentHashMap 支持 CurrencyLevel (Segment 数组数量)的线程并发。每当一个线程占用锁访问一个 Segment 时，不会影响到其他的 Segment。

#### put方法

> 1. 将当前 Segment 中的 table 通过 key 的 hashcode 定位到 HashEntry。
> 2. 遍历该 HashEntry，如果不为空则判断传入的 key 和当前遍历的 key 是否相等，相等则覆盖旧的 value。
> 3. 不为空则需要新建一个 HashEntry 并加入到 Segment 中，同时会先判断是否需要扩容。
> 4. 最后会解除在 1 中所获取当前 Segment 的锁。



#### get方法

> 只需要将 Key 通过 Hash 之后定位到具体的 Segment ，再通过一次 Hash 定位到具体的元素上。
>
> 由于 HashEntry 中的 value 属性是用 volatile 关键词修饰的，保证了内存可见性，所以每次获取时都是最新值。
>
> ConcurrentHashMap 的 get 方法是非常高效的，**因为整个过程都不需要加锁**。



#### 1.8新特性

抛弃了Segment分段锁，使用CAS+Synchronized保证并发安全。

> 1.8 在 1.7 的数据结构上做了大的改动，采用红黑树之后可以保证查询效率（`O(logn)`），甚至取消了 ReentrantLock 改为了 synchronized，这样可以看出在新版的 JDK 中对 synchronized 优化是很到位的。

#### put方法

- 根据 key 计算出 hashcode 。
- 判断是否需要进行初始化。
- `f` 即为当前 key 定位出的 Node，如果为空表示当前位置可以写入数据，利用 CAS 尝试写入，失败则自旋保证成功。
- 如果当前位置的 `hashcode == MOVED == -1`,则需要进行扩容。
- 如果都不满足，则利用 synchronized 锁写入数据。
- 如果数量大于 `TREEIFY_THRESHOLD` 则要转换为红黑树。

#### get方法

- 根据计算出来的 hashcode 寻址，如果就在桶上那么直接返回值。
- 如果是红黑树那就按照树的方式获取值。
- 就不满足那就按照链表的方式遍历获取值。
