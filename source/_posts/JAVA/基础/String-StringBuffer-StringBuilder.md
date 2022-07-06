---
title: String、StringBuffer、StringBuilder
date: 2022-07-06 09:21:37
categories:
- JAVA
- 基础
tag:
- 面试
top:
keywords:
description:
cover: 'https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207060926559.png'
top_img: 'https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207060926559.png'
---

### String、StringBuffer、StringBuilder

1. String：只读字符串；其实就是对象，但是底层是被**final修饰**的字符数组，引用的字符串不能改变；因此每次操作都是新的String的对象。
2. StringBuffer和StringBuilder：底层都是可变的字符数组，所以在进行频繁的字符串操作时，建议使用StringBuffer和 StringBuilder来进行操作。 
3. StringBuffer 对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的。
4. StringBuilder 并没有对方法进行加同步锁，所以是非线程安全的。



### String理解

1. java.lang包；类中包含有对字符串的操作方法

2. 字符串的值创建后不能更改（String是一种引用类型，String的值在堆内存中，）

   - 字符串：其对象的引用都是存储在栈中的，如果是编译期已经创建好(直接用双引号定义的)的就存储在常量池中，如果是运行期（new出来的）才能确定的就存储在堆中。
   - 对于equals相等的字符串，在常量池中永远只有一份，在堆中有多份。String分配很耗内存；就用常量池分配，如果常量池中存在就堆中的String就指向这个值的内存地址。

3. String类是final类，也即意味着String类不能被继承，并且它的成员方法都默认为final方法。在Java中，被final修饰的类是不允许被继承的，并且该类中的成员方法都默认为final方法。

4. 上面列举出了String类中所有的成员属性，从上面可以看出String类其实是通过char数组来保存字符串的。

5. **"=="比较的是内存地址，"equals"比较的是值**

   ==**补充说明：Java String 和 new String()的区别**==

   　　**栈区存引用和基本类型，不能存对象，而堆区存对象。==是比较地址，equals()比较对象内容。**

   (1) **String str1 = "abcd"的实现过程**：首先栈区创建str引用，**然后在String池（独立于栈和堆而存在，存储不可变量）中寻找其指向的内容为"abcd"的对象，如果String池中没有，则创建一个，然后str指向String池中的对象，如果有，则直接将str1指向"abcd""；**如果后来又定义了字符串变量 str2 = "abcd",则直接将str2引用指向String池中已经存在的“abcd”，不再重新创建对象；当str1进行了赋值（str1=“abc”），则str1将不再指向"abcd"，而是重新指String池中的"abc"，此时如果定义String str3 = "abc",进行str1 == str3操作，返回值为true，因为他们的值一样，地址一样，但是如果内容为"abc"的str1进行了字符串的+连接str1 = str1+"d"；此时str1指向的是在堆中新建的内容为"abcd"的对象，即此时进行str1==str2，返回值false，因为地址不一样。

   **(2) String str3 = new String("abcd")的实现过程**：**直接在堆中创建对象。**如果后来又有String str4 = new String("abcd")，str4不会指向之前的对象，而是重新创建一个对象并指向它，所以如果此时进行str3==str4返回值是false，因为两个对象的地址不一样，如果是str3.equals(str4)，返回true,因为内容相同。



### String拼接字符串

String 是 Java 中一个不可变的类，所以他一旦被实例化就无法被修改。

不可变类的实例一旦创建，其成员变量的值就不能被修改。这样设计有很多好处，比如 可以缓存 hashcode、使用更加便利以及更加安全等。

所以拼接就是创建一个新字符串，然后拼接。

```java
String s = "abcd";
s = s.concat("ef");
```

![image-20220602231927181](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202207060926559.png)

`s 中保存的是一个重新创建出来的 String 对象的引用。 `

“+”：起始就是语法糖让我们的程序更简洁，可读性更好。

Concat字符串拼接方法。

原理：

> 创建了一个字符数组，长度是已有字符串和待拼接字符串的长度之和，再 把两个字符串的值复制到新的字符数组中，并使用这个字符数组创建一个新的 String 对象 并返回。（本质也是new String对象返回）

```java
public String concat(String str)
int otherLen = str.length();
if (otherLen == 0) {
return this;
}
int len = value.length;
char buf[] = Arrays.copyOf(value, len + otherLen);
str.getChars(buf, len);
return new String(buf, true);
}
```



StringBuffer和StringBuilder对字符串进行拼接。

```java
StringBuffer wechat = new StringBuffer("Hollis");
String introduce = "每日更新 Java 相关技术文章";
StringBuffer hollis = wechat.append(",").append(introduce);

StringBuilder wechat = new StringBuilder("Hollis");
String introduce = "每日更新 Java 相关技术文章";
StringBuilder hollis = wechat.append(",").append(introduce);
```

JAVA8提供了String类的jion的静态方法和StringsUtils.join；

StringUtils 中提供的 join 方法，最主要的功能是：将数组或集合以 某拼接符拼接到一起形成新的字符串

```java
String wechat = "Hollis";
String introduce = "每日更新 Java 相关技术文章";
System.out.println(StringUtils.join(wechat, ",", introduce));

String []list ={"Hollis","每日更新 Java 相关技术文章"};
String result= StringUtils.join(list,",");
System.out.println(result);
//结果：Hollis,每日更新 Java 相关技术文章
```

#### StringUtils.join 是如何实现拼接

```java
public static String join(final Object[] array, String separator, final int startIndex, final int endIndex) {
	if (array == null) {
		return null;
	}
	if (separator == null) {
		separator = EMPTY;
	}
// endIndex - startIndex > 0: Len = NofStrings *(len(firstString) + len(separator))
// (Assuming that all Strings are roughly equally long)
	final int noOfItems = endIndex - startIndex;
	if (noOfItems <= 0) {
		return EMPTY;
	}
	final StringBuilder buf = new StringBuilder(noOfItems * 16);
	for (int i = startIndex; i < endIndex; i++) {
		if (i > startIndex) {
			buf.append(separator);
		}
		if (array[i] != null) {
			buf.append(array[i]);
		}
	}
	return buf.toString();
}
```

`看出就是通过StringBuilder的append方法实现的`

#### StringBuffer 的 append 方法。 

```java
public synchronized StringBuffer append(String str) {
	toStringCache = null;
	super.append(str);
	return this;
}
```

#### 总结

**Java 开发手册建议：循环体内，字符串的连接方式，使用 StringBuilder 的 append 方法进行扩展。而不要使用+。**





### 