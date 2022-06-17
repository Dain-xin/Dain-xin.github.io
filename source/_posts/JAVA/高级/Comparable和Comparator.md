---
title: Comparable和Comparator两个接口
date: 2022-06-17 14:40:37
categories:
- JAVA
- 高级
---

# Comparable

a、Comparable是java.lang包下面的接口，lang包下面可以看做是java的基础语言接口。

b、强行对实现它的每个类的对象进行整体排序。这种排序被称为类的自然排序，类的compareTo方法 被称为它的自然比较方法。只能在类中实现compareTo()一次，不能经常修改类的代码实现自己想要的排序。

c、实现 此接口的对象列表（和数组）可以通过Collections.sort（和Arrays.sort）进行自动排序，对象可以用作有序映射中 的键或有序集合中的元素，无需指定比较器。

```java
Collections.sort();//源码

public static<T extends Comparable<? super T> > void sort(List <T> list){

list.sort(null);

}    

 

List-->sort()

default void sort(Comparator<?superE>c){

Object[]a=this.toArray();

Arrays.sort(a,(Comparator)c);

ListIterator<E> i=this.listIterator();

for(Objecte:a){

	i.next();

	i.set((E)e);

}

}


```

 

注：根据元素的*自然顺序* 对指定列表按升序进行排序。列表中的所有元素都必须实现 Comparable 接口。此外，列表中的所有元素都必须是*可相互比较的*（也就是说，对于列表中的任何 e1 和 e2 元素，e1.compareTo(e2) 不得抛出 ClassCastException）。

 

d、当使用这些可排序的集合添加相应的对象时，就会调用  compareTo方法来进行natural ordering（自然排序）的排序。

e、几乎所有的数字类型对象：Integer, Long，Double等包装类都实现了这个Comparable接口。  也就有了compareTo的方法。 

f、**ClassCastException：当试图将对象强制转换为不是实例的子类时，抛出该异常。**

**理解：没有可比性的两个对象就会报错。**

 

# Comparator（util包）

 

a、public interface **Comparator<T>：** **T -** **此** **Comparator** **可以比较的对象类型**

代表其是一个工具类，用来辅助排序的

b、强行对某个对象 collection 进行*整体排序* 的比较函数。可以将 Comparator 传递给 sort 方法（如 Collections.sort) 或 Arrays.sort），从而允许在排序顺序上实现精确控制

c、Comparator是一个Functional Interface功能接口，需要实现compare方法：

重写方法：

| int  compare(T o1, T o2) ： | 比较用来排序的两个参数 |
| --------------------------- | ---------------------- |
|                             |                        |

 

d、Collections的sort() ：

public static <T> void **sort**([List](mk:@MSITStore:E:\_1_NewFile\$_好用软件&工具\_API工具\API1_6.CHM::/java/util/List.html)<T> list, [Comparator](mk:@MSITStore:E:\_1_NewFile\$_好用软件&工具\_API工具\API1_6.CHM::/java/util/Comparator.html)<? super T> c)

注：

1、根据指定比较器产生的顺序对指定列表进行排序。**此列表内的所有元素都必须可使用指定比较器****相互比较**（也就是说，对于列表中的任意 e1 和 e2 元素，c.compare(e1, e2) 不得抛出 ClassCastException）。此排序被保证是*稳定的*：不会因调用 sort 而对相等的元素进行重新排序。

2、Collections.sort(List,Comparator),Arrays.sort(Object[],Comparator) 都是这个。

 

 

### **对Comparable**和**Compatator**接口的理解：

1、我们提到**Comparable**指定了对象的**natural ordering**，如果我们在添加到可排序集合类的时候想按照我们**自定义的方式进行排序**，这个时候就需要使用到**Comparator**了。

2、在排序过程中，首先会去检查Comparator是否存在，如果不存在则会使用默认的natural ordering。

3、Comparator允许对null参数的比较，而Comparable是不允许的，否则会爬出NullPointerException。

**4**、**Comparatable**和**Comparator**的例子：

eg:

```java
List<Integer>list1=Arrays.asList(5,3,2,4,1);

Collections.sort(list1,newComparator<Integer>(){

@Override
public int compare(Integero1,Integero2){
    if(o1>o2)
        return -1;
    else if(o1==o2){
        return 0;
    }return 1;
}
});//降序

Collections.sort(list1);
```

 

默认情况下Integer是按照升序来排的，但是我们可以通过传入一个Comparator来改变这个过程变成降序。