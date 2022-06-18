---
title: JAVA反射-Reflection
date: 2022-06-18 23:03:37
categories:
- JAVA
- 高级
- 反射和代理
---

# [反射 - 廖雪峰](https://www.liaoxuefeng.com/wiki/1252599548343744/1255945147512512)

- Reflection（反射）是被视为动态语言的关键，反射机制允许程序在执行期     借助于Reflection     API取得任何类的内部信息，并能直接操作任意对象的内部属性及方法。加载完类之后，在堆内存的方法区中就产生了一个Class类型的对象（一个     类只有一个Class对象），这个对象就包含了完整的类的结构信息。我们可以通过这个对象看到类的结构。这个对象就像一面镜子，透过这个镜子看     到类的结构，我们形象的称之为：**反射**。

正常方式： 引入需要的”包类”名称 ---> 通过new实例化 ---> 取得实例化对象

反射方式： 实例化对象 --->    getClass()方法 ---> 得到完整的“包类”名称

##  **反射需要理解的知识点：**

- 如何反射获取 Class 对象
- 如何反射获取类中的所有字段
- 如何反射获取类中的所有构造方法
- 如何反射获取类中的所有非构造方法

**注：**

**1****、测试时可以利用反射 API 访问类的私有成员，以保证测试代码覆盖率。**

**2****、很多程序架构，尤其是三方框架，无法保证自己的封装是完美的。**

**3****、如果没有反射，对于外部类的私有成员，我们将一筹莫展，所以我们有了反射这一后门，为程序设计提供了更大的灵活性。**

 

**补充：**

## **动态语言  vs 静态语言**

**1、动态语言**

**是一类在运行时可以改变其结构的语言：例如新的函数、对象、甚至代码可以 被引进，已有的函数可以被删除或是其他结构上的变化。通俗点说就是****在运 行时代码可以根据某些条件改变自身结构。

**主要动态语言：****Object-C、C#、JavaScript、PHP、Python、Erlang。

**2、静态语言**

**与动态语言相对应的，****运行时结构不可变的语言就是静态语言。如Java、C、**C++。**

**注：**

**Java不是动态语言，但Java可以称之为“准动态语言”**。

**Java有一定的动 态性，我们可以利用反射机制、字节码操作获得类似动态语言的特性。**

**Java的动态性让编程的时候更加灵活！**

 

# **反射**API

## API成员

Java 类的成员包括以下三类：属性字段、构造函数、方法。反射的 API 也是与这几个成员相关：

![image-20220618231056919](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206182310128.png)

- Field 类：提供有关类的属性信息，以及对它的动态访问权限。它是一个封装反射类的属性的类。
- Constructor 类：提供有关类的构造方法的信息，以及对它的动态访问权限。它是一个封装反射类的构造方法的类。
- Method 类：提供关于类的方法的信息，包括抽象方法。它是用来封装反射类方法的一个类。
- Class 类：表示正在运行的 Java 应用程序中的类的实例。
- Object 类：Object 是所有 Java 类的父类。所有对象都默认实现了 Object 类的方法。

## **class**类加载的过程：

![image-20220618231309218](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206182313338.png)

### 加载-链接-初始化

![image-20220618231353606](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206182313821.png)

## **Class**类（lang包）

 

```java
//获取 Class 对象有三种方式：

//1.**通过字符串获取**Class**对象，这个字符串必须带上完整路径名*

Class studentClass = Class.*forName*("com.test.reflection.Student");

//2.**通过类的**class**属性*

Class studentClass2 = Student.class;

//3.**通过对象的**getClass()**函数*

Student studentObject = new Student();

Class studentClass3 = studentObject.getClass();
```

**理解**：

第一种方法是通过类的全路径字符串获取 Class 对象，这也是我们平时最常用的反射获取 Class 对象的方法；

第二种方法有限制条件：需要导入类的包；

第三种方法已经有了 Student 对象，不再需要反射。

三种方法得到的都是同一个对象，且java运行中只会生成一个Class对象



### Class实例比较和instance of的差别：

```java
		Integer n = new Integer(123);
		boolean b1 = n instance of Integer; 
// true，因为n是Integer类型
		boolean b2 = n instanceof Number; 
// true，因为n是Number类型的子类
		boolean b3 = n.getClass() == Integer.class; 
// true，因为n.getClass()返回Integer.class
		boolean b4 = n.getClass() == Number.class; 
// false，因为Integer.class!=Number.class

```

### **CLass**对象理解：

JVM为每个加载的class及interface创建了对应的Class实例来保存class及interface的所有信息；

获取一个class对应的Class实例后，就可以获取该class的所有信息；

通过Class实例获取class信息的方法称为反射（Reflection）；

JVM总是动态加载class，可以在运行期根据条件来控制加载class。

## ClassLoader理解

![image-20220618231944648](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206182319971.png)

### [Java类加载原理解析](https://blog.csdn.net/justloveyou_/article/details/72217806)

**三种类加载器：**

- **启动（Bootstrap）类加载器：引导类装入器是用本地代码实现的类装入器，它负责将     <Java_Runtime_Home>/lib下面的核心类库或-Xbootclasspath选项指定的jar包加载到内存中。由于引导类加载器涉及到虚拟机本地实现细节，开发者无法直接获取到启动类加载器的引用，所以不允许直接通过引用进行操作。**
- **扩展（Extension）类加载器：扩展类加载器是由Sun的ExtClassLoader（sun.misc.Launcher$ExtClassLoader）实现的。它负责将<     Java_Runtime_Home     >/lib/ext或者由系统变量-Djava.ext.dir指定位置中的类库加载到内存中。开发者可以直接使用标准扩展类加载器。**
- **系统（System）类加载器：系统类加载器是由     Sun的 AppClassLoader（sun.misc.Launcher$AppClassLoader）实现的。它负责将系统类路径java     -classpath或-Djava.class.path变量所指的目录下的类库加载到内存中。开发者可以直接使用系统类加载器。**
- **除了以上列举的三种类加载器，还有一种比较特殊的类型就是线程上下文类加载器，这个将在后面单独介绍。**

## **访问字段：**

**得到的**Class**对象就等于获取到了所有信息。**

| **Field getField(name)：**         | **根据字段名获取某个public的field（包括父类）**              |
| ---------------------------------- | ------------------------------------------------------------ |
| **Field getDeclaredField(name)：** | **根据字段名获取当前类的某个（包括private**）field（不包括父类） |
| **Field[] getFields()：**          | **获取所有public的field（包括父类）**                        |
| **Field[] getDeclaredFields()：**  | **获取当前类的所有field（不包括父类）**                      |

 

Classs tdClass=Student.class;

*//**获取**public**字段**"score":*

System.*out*.println(stdClass.getField("score"));

*//**获取继承的**public**字段**"name":*

System.*out*.println(stdClass.getField("name"));

*//**获取**private**字段**"grade":*

System.*out*.println(stdClass.getDeclaredField("score"));



**如果没有就报：** **java.lang.NoSuchFieldException: score**

 

### **Field**类（**java.lang.reflect**）

- getName()：返回字段名称，例如，"name"；

- getType()：返回字段类型，也是一个Class实例，例如，String.class；

- getModifiers()：返回字段的修饰符，它是一个int，不同的int表示不同的含义。

- - 代表了：public 、private 、final 、protected 、static…..
  - Modifier.*toString*(constructor.getModifiers())：返回对应的修饰符

 

### **Modifier类（java.lang.**reflect）

提供了 static 方法和常量，对类和成员访问修饰符进行解码。

修饰符集被表示为整数，用不同的位位置 (bit position) 表示不同的修饰符。

 

| static String toString(int mod) | 返回描述指定修饰符中的访问修饰符标志的字符串。 |
| ------------------------------- | ---------------------------------------------- |
|                                 |                                                |

 

 

## **设置字段：**

1、调用Field.setAccessible(true)的意思是，别管这个字段是不是public，一律允许访问。

Field.get(Object)获取指定实例的指定字段的值。

 

注：修改非public字段，需要首先调用setAccessible(true)。

2、可以获取指定实例的字段值，也可以设计字段值。

Field.set(Object, Object)实现的，其中第一个Object参数是指定的**实例**，第二个Object参数是**待修改的值（属性）**。

eg：

```java
Class<?>clazz=Class.*forName*("com.java_2_advanced.test16_reflection.Reflection1");

String simpleName=clazz.getSimpleName();

System.*out*.println(simpleName);

Object object=clazz.newInstance();

Field name=clazz.getDeclaredField("name");

Field age=clazz.getDeclaredField("age");

name.setAccessible(true);

name.set(object,"djx");

System.*out*.println(name.get(object));

age.set(object,"18");

System.*out*.println(age.get(object));

 
```

## **访问构造方法**

![image-20220618232749333](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206182327707.png)

## **Class**对象可以获取继承关系：

- Class     getSuperclass()：获取父类类型；
- Class[]     getInterfaces()：获取当前类实现的所有接口。

通过Class对象的isAssignableFrom()方法可以判断一个向上转型是否可以实现。

```java
Class claxx=ArrayList.class;

Class[] interfaces=claxx.getInterfaces();

System.*out*.println("ArrayList的接口：");

for(inti=0;i<interfaces.length;i++){

System.*out*.println(interfaces[i].getSimpleName());

}

Class superclass=claxx.getSuperclass();

System.*out*.println("ArrayList父类:");

System.*out*.println(superclass.getName());
```

结果：

![](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206182343437.png)

访问方法：

| public Method[] getDeclaredMethods() | 返回此Class对象所表示的类或接口的全部方法     |
| ------------------------------------ | --------------------------------------------- |
| public Method[] getMethods()         | 返回此Class对象所表示的类或接口的public的方法 |

```java
//**方法调用*

Method printIt1=clazz.getDeclaredMethod("printIt1");

Object o1=clazz.newInstance();

printIt1.invoke(o1);

Method printIt2=clazz.getDeclaredMethod("printIt2",String.class);

System.*out*.println(printIt2.invoke(o1,"djx2"));

Method printIt3=clazz.getDeclaredMethod("printIt3",int.class,String.class);

printIt3.invoke(o1,18,"djx3");
```

![image-20220618234459952](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206182345058.png)

 

**Method**类中：（java.lang.**reflect** **）**

| **public Class<?> getReturnType()**       | **取得全部的返回值** |
| ----------------------------------------- | -------------------- |
| **public Class<?>[] getParameterTypes()** | **取得全部的参数**   |
| **public int getModifiers()**             | **取得修饰符**       |
| **public Class<?>[] getExceptionTypes()** | **取得异常信息**     |

## **代理方法**

![image-20220618234741970](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206182347246.png)