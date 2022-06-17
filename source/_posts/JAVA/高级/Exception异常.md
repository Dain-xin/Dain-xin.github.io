---
title: Exception异常
date: 2022-06-17 17:40:37
categories:
- JAVA
- 高级
---

# **异常根类**

Throwable**（lang包）：两个子类：Error（lang包）和Exception（lang包）：**

1、根类的子类（**Throwable**类）：

- **error---错误** ： 是指程序无法处理的错误，表示应用程序运行时出现的重大错误。（**只能避免**）

例如jvm运行时出现的OutOfMemoryError以及Socket编程时出现的端口占用等程序无法处理的错误。

- **Exception     --- 异常**     ：异常可分为运行时异常跟编译异常，指的是程序在执行过程中，出现的非正常的情况，最终会导致JVM的非正常停止。（**可以避免，直接处理**）

2、根类方法：

| void printStackTrace() | 打印异常的详细信息。  包含了异常的类型,异常的原因,还包括异常出现的位置,在开发和调试阶段,都得使用printStackTrace。 |
| ---------------------- | ------------------------------------------------------------ |
| String getMessage()    | 获取发生异常的原因。 提示给用户的时候,就提示错误原因。       |
| String toString()      | 获取异常的类型和异常描述信息(不用)。                         |

 

3、异常Exception和错误Error子类：

分类：

## 运行时异常（RuntimeException）

常见：

如 **NullPointerException、IndexOutOfBoundsException** 等，这些异常是不检查异常，程序中可以选择捕获处理，也可以不处理。这些异常一般由程序逻辑错误引起，程序应该从逻辑角度尽可能避免这类异常的发生。

 

## 编译时异常（RuntimeException以外的异常（Check异常））

常见：

从程序语法角度讲是必须进行处理的异常，如果不处理，程序就不能编译通过。如 **IOException、ClassNotFoundException**(加载类出现异常)、ParseException（解析时出现意外错误）等以及用户自定义的 Exception 异常。

 

图示列举：

 java.lang 中定义的运行时异常和非运行时异常的类型及作用。

##  常见异常

**表 1 Java中常见运行时异常：**

| **异常类型**                  | **说明**                                                     |
| ----------------------------- | ------------------------------------------------------------ |
| ArithmeticException           | 算术错误异常，如以零做除数                                   |
| ArraylndexOutOfBoundException | 数组索引越界                                                 |
| ArrayStoreException           | 向类型不兼容的数组元素赋值                                   |
| ClassCastException            | 类型转换异常                                                 |
| IllegalArgumentException      | 使用非法实参调用方法                                         |
| lIIegalStateException         | 环境或应用程序处于不正确的状态                               |
| lIIegalThreadStateException   | 被请求的操作与当前线程状态不兼容                             |
| IndexOutOfBoundsException     | 某种类型的索引越界                                           |
| **NullPointerException**      | 尝试访问 null 对象成员，空指针异常（**指的是对象为空的异常，静态方法不需要实例**） |
| NegativeArraySizeException    | 再负数范围内创建的数组                                       |
| NumberFormatException         | 数字转化格式异常，比如字符串到 float 型数字的转换无效        |
| TypeNotPresentException       | 类型未找到                                                   |

 

**表 2 Java常见非运行时异常：**

| **异常类型**                 | **说明**                   |
| ---------------------------- | -------------------------- |
| ClassNotFoundException       | 没有找到类                 |
| IllegalAccessException       | 访问类被拒绝               |
| InstantiationException       | 试图创建抽象类或接口的对象 |
| InterruptedException         | 线程被另一个线程中断       |
| NoSuchFieldException         | 请求的域不存在             |
| NoSuchMethodException        | 请求的方法不存在           |
| ReflectiveOperationException | 与反射有关的异常的超类     |
| IOException                  | IO 操作异常                |

 

 

**表** **3** **错误（Error）：**

| NoClassDefFoundError： | 找不到 class 定义异常          |
| ---------------------- | ------------------------------ |
| StackOverflowError：   | 深递归导致栈被耗尽而抛出的异常 |
| OutOfMemoryError：     | 内存溢出异常                   |

 

 

### 异常处理：

理解：

java处理异常的机制：抛出异常以及捕获异常 ，一个方法所能捕捉的异常，一定是Java代码在某处所抛出的异常。

**简单地说，异常总是先被抛出，后被捕捉的。如果自己不处理，交给****JVM****处理，就会系统中断。**

 

如何捕获异常：

1）首先java对于异常捕获使用的是try---catch或try --- catch --- finally 代码块，程序会捕获try代码块里面的代码，

若捕获到异常则进行catch代码块处理。若有**finally则在catch处理后执行finally里面的代码**。

然而存在这样两个问题：

（a.看如下代码：

```java
try{
   //待捕获代码
 }catch（Exception e）{
   System.out.println("catch is begin");
   return 1 ；
 }finally{
   System.out.println("finally is begin");
 }
```

在catch里面有一个return，那么finally还是会被执行

catch is begin
 finally is begin 

 

解释：也就是说会先执行catch里面的代码后执行finally里面的代码最后才return 1 ；

 

（b.看如下代码：

```java
try{
  //待捕获代码  
 }catch（Exception e）{
   System.out.println("catch is begin");
   return 1 ；
 }finally{
   System.out.println("finally is begin");
   return 2 ;
 }

 
```

**注意：**

**在b代码中输出结果跟a是一样的，然而返回的是return 2 ； 原因很明显，就是执行了finally后已经return了，所以catch里面的return不会被执行到。也就是说finally永远都会在catch的return前被执行。（这个是面试经常问到的问题哦！）**

 

 

 

# **throw跟throws的区别:**

 

```java
public void test() throws Exception {
   throw new Exception();
 }
```

从上面这一段代码可以明显的看出两者的区别。

**throws表示一个方法声明可能抛出一个异常，throw表示此处抛出一个已定义的异常（可以是自定义需继承Exception，也可以是java自己给出的异常类）。**



注：

1. **catch     不能独立于 try 存在。**
2. **在 try/catch 后面添加 finally 块并非强制性要求的。**
3. **try 代码后不能既没 catch 块也没 finally 块。**
4. **try, catch, finally 块之间不能添加任何代码。**

 

 

# 异常的注意点

- 捕获多个异常时Exception的捕获只能放最后，会影响前面指定的特殊的异常处理，（可以理解为处理顺序进行）；父类Exception e在前面，南无后面的子类异常就不会处理（报错）。

- try{  } 包裹的内容，如果遇到异常，就会跳到catch（）{      } 的处理，原来try的异常后面的代码就不会在运行了。

 

代码：

```java
try{

int[] arr={1,2,3,4};

System.*out*.println(arr[6]);

String s=null;

System.*out*.println(s.length());

System.*out*.println("这里不会执行");

} catch(ArrayIndexOutOfBoundsException e){

System.*err*.println("下标超界异常");

} catch(NullPointerException e){

System.*err*.println("空指指针异常");

} finally{

System.*out*.println("finally");

}

System.*out*.println("execute over");
```

 

结果：![image-20220617191651449](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206171916511.png)

 

- 而且在catch里处理异常的时候不要简单的e.printStackTrace()，而是**应该进行详细的处理。比如进行console打印详情或者进行日志记录。**

- **throws** **抛异常，父类可以抛出多个异常，但是不能既有父类异常还包含子类异常，**

![image-20220617191746639](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206171917705.png)

![image-20220617191759250](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206171917330.png)