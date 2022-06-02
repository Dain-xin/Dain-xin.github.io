---
title: Java基础——重难点
date: 2022-05-25 21:47:17
categories:
- JAVA
- 基础
tags:
- 面试
---

### 面向过程和面向对象

A：面向过程：

概述: 自顶而下的编程模式 把问题分解成一个一个步骤，每个步骤用函数实现，依次调用即可。 

就是说，在进行面向过程编程的时候，不需要考虑那么多，上来先定义一个函数，然后 使用各种诸如 if-else、for-each 等方式进行代码执行。 

最典型的用法就是实现一个简单的算法，比如实现冒泡排序。 

B：面向对象：

概述: 将事务高度抽象化的编程模式 将问题分解成一个一个步骤，对每个步骤进行相应的抽象，形成对象，通过不同对象之 间的调用，组合解决问题。 具有封装、继承、多态特点——低耦合-高内聚。

就是说，在进行面向对象进行编程的时候，要把属性、行为等封装成对象，然后基于这 些对象及对象的能力进行业务逻辑的实现。 

比如：想要造一辆车，上来要先把车的各种属性定义出来，然后抽象成一个 Car 类

### hashCode作用

理解：

它返回的就是根据对象的内存地址换算出的一个值。这样一来，当 集合要添加新的元素时，先调用这个元素的hashCode方法，就一下子能定位到它应该放置的物理 位置上。

如果这个位置上没有元素，它就可以直接存储在这个位置上，不用再进行任何比较了；

如果这个位置上已经有元素了，就调用它的equals方法与新元素进行比较，相同的话就不存了，不相 同就散列其它的地址。

这样一来实际调用equals方法的次数就大大降低了，几乎只需要一两次。

### JAVA语言特点

1. 简单易学、有丰富的类库；
2. 面向对象、低耦合，高内聚；
3. 跨平台特点（JVM是跨平台的根本）；
4. 安全可靠、支持多线程。

### 基本数据类型和包装类

1. int默认值是0，而Integer默认值 是null，所以Integer能区分出0和null的情况；
2. 包装类可以作为泛型，实现了Serializable接口，支持序列化和反序列化
3.  Integer 来说，在数值区间为 -128~127 时，会直接复用已有对象，在这区间之外的数字才会在堆上产生。例如128这个会生成新对象。
4. 包装类包含了丰富的方法。

### 重写和重载的理解

重写：（Overread）

1. 子类重写父类方法发生在父类与子类之间 
2. 方法名，参数列表，返回类型（除过子类中方法的返回类型 是父类中返回类型的子类）都必须相同 
3. 访问修饰符的限制一定要大于被重写方法的访问修饰符 （public>protected>default>private) 
4. 重写方法一定不能抛出新的检查异常或者比被重写方法申 明更加宽泛的检查型异常

重载：（Overload）

1. 重载Overload是一个类中多态性的一种表现 
2. 重载要求同名方法的参数列表不同(参数类型，参数个数甚至是参数顺序)
3. 重载的时候，返回值类型可以相同也可以不相同。无法以返回 型别作为重载函数的区分标准

### Instanceof 关键字

Instanceof 严格来说是Java中的一个双目运算符，用来测试一个对象是否为一个类的实例；也就是：

```java
boolean result = obj instanceof Class
/*
其中 obj 为一个对象，Class 表示一个类或者一个接口，当 obj 为 Class 的对象，
或者是直接、间接子类、其接口的实现类，结果result 都返回 true，否则返回false。

原理：编译器会检查 obj 是否能转换成右边的class类型，如果不能转换则直接报错。
*/
```

### String、StringBuffer、StringBuilder

1. String：只读字符串；其实就是对象，但是底层是被**final修饰**的字符数组，引用的字符串不能改变；因此每次操作都是新的String的对象。
2. StringBuffer和StringBuilder：底层都是可变的字符数组，所以在进行频繁的字符串操作时，建议使用StringBuffer和 StringBuilder来进行操作。 
3. StringBuffer 对方法加了同步锁或者对调用的方法加了同步锁，所以是线程安全的。
4. StringBuilder 并没有对方法进行加同步锁，所以是非线程安全的。

### String拼接字符串

String 是 Java 中一个不可变的类，所以他一旦被实例化就无法被修改。

不可变类的实例一旦创建，其成员变量的值就不能被修改。这样设计有很多好处，比如 可以缓存 hashcode、使用更加便利以及更加安全等。

所以拼接就是创建一个新字符串，然后拼接。

```java
String s = "abcd";
s = s.concat("ef");
```

![image-20220602231927181](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206022319227.png)

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

### 语法糖

[Java常见的12个语法糖](https://blog.csdn.net/coder_what/article/details/96310967?utm_medium=distribute.pc_relevant.none-task-blog-baidujs_baidulandingword-1&spm=1001.2101.3001.4242)

语法糖往往给程序员**提供了更实用的编码方式，有益于更好的编码风格，更易读**。不过其并没有给语言添加什么新东西。需要声明的是“语法糖”这个词绝非贬义词，它可以给我带来方便，是一种便捷的写法，编译器会帮我们做转换；而且可以提高开发编码的效率，在性能上也不会带来损失。

**常见的语法糖：**

1、switch、 case、default 。

2、for-each：其内部仍然是采用普通for循环和迭代器实现遍历。

3、Lambda表达式：函数式编程。

4、enum枚举。

5、泛型

6、多态

7、内部类

8、自动封装

9、变长参数（…的方法）

10、if语句

11、try-with-resource

12、编译时优化：主要针对常量。

### 常量池

在 Java 体系中，共用三种常量池。分别是字符串常量池、Class 常量池 和运行时常量池。

####  字符串常量池

在 JVM 中，为了减少相同的字符串的重复创建，为了达到节省内存的目的。会单独开 辟一块内存，用于保存字符串常量，这个内存区域被叫做字符串常量池。

字符串常量池的位置：永久代中。

#### Class常量池

Class 常量池可以理解为是 Class 文件中的资源仓库。Class 文件中除了包含类的版 本、字段、方法、接口等描述信息外，还有一项信息就是常量池(constant pool table)， 用于**存放编译器生成的各种字面量(Literal)和符号引用(Symbolic References)。** 

由于不同的 Class 文件中包含的常量的个数是不固定的，所以在 Class 文件的常量池 入口处会设置两个字节的常量池容量计数器，记录了常量池中常量的个数。

![image-20220603005157329](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206030051366.png)

[00-Java开发基础.pdf](file:///E:/_1_NewFile/_$_Java后端开发必备手册/JAVA面试/00-Java开发基础.pdf)  90页
