---
title: Object类
date: 2022-06-16 10:52:37
categories:
- JAVA
- 高级
---

# Object类：（**lang**包）

1、由于 Java 所有的类都是 Object 类的子类，所以任何 Java 对象都可以调用 Object 类的方法。

里面：静态方法；因此也不能继承。是所有类的父类（超类）。

##  **Object 类的常用方法**

**方法说明：** 

- Object clone()    创建与该对象的类相同的新对象（复制作用）

- boolean equals(Object)  比较两对象是否相等（对象非空|数据类型一致）默认比较地址

```java
public boolean equals(Object o){

if(this==o)return true;

if(o==null||getClass()!=o.getClass())return false;

Object1 object1=(Object1)o;

return Objects.equals(name , object1.name);

}
```

 

- String toString()  返回该对象的字符串表示

​		由于toString方法返回的结果是内存地址，所以往往重写来输出想要的属性数据

 

- Class getClass()  返回一个对象运行时的实例类

- int hashCode()  返回该对象的散列码值

  ```java
  System.out.println("ABCDEa123abc".hashCode()); // 165374702
  
  System.out.println("ABCDFB123abc".hashCode()); // 165374702
  
  //有的字符串hashcode值一样。
  ```

- void finalize()   当垃圾回收器确定不存在对该对象的更多引用时，对象垃圾回收器调用该方法
- void notify()  激活等待在该对象的监视器上的一个线程
- void notifyAll() 激活等待在该对象的监视器上的全部线程
- void wait()  在其他线程调用此对象的 notify() 方法或 notifyAll() 方法前，导致当前线程等待

 

## Objects类（util包）

a、在JDK7添加了一个Objects工具类，它提供了一些方法来操作对象，它由一些静态的实用方法组成。

b、这些方法解决了Object类的空指针问题。在比较两个对象的时候，Object的equals方法容易抛出空指针异常，而Objects类中的equals方法就优化了这个问题。

eg：

```java
public static boolean equals(Object a, Object b) {
	return (a == b) || (a != null && a.equals(b));
}
```

注：其实可以重写的时候在即添加对象为空的判断即可。

4、String类中的equals方法是用来判断两个对象的内容是否相同，

而Object 类中的equals方法是用来判断两个对象是否是同一个对象，**所谓同一个对象指的是内存中的同一块存储空间。**