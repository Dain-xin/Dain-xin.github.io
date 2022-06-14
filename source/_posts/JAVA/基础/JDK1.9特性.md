---
title: JDK1.9特性
date: 2022-06-14 08:26:52
categories:
- JAVA
- 基础
---



# JDK9

最主要的变化是已经实现的模块化系统

|  1   | [模块系统](https://www.runoob.com/java/java9-module-system.html) |
| :--: | :----------------------------------------------------------: |
|  2   | [REPL (JShell)](https://www.runoob.com/java/java9-repl.html) |
|  3   | [改进的 Javadoc](https://www.runoob.com/java/java9-improved-javadocs.html) |
|  4   | [多版本兼容 JAR 包](https://www.runoob.com/java/java9-multirelease-jar.html) |
|  5   | [集合工厂方法](https://www.runoob.com/java/java9-collection-factory-methods.html) |
|  6   | [私有接口方法](https://www.runoob.com/java/java9-private-interface-methods.html) |
|  7   | [进程 API](https://www.runoob.com/java/java9-process-api-improvements.html) |
|  8   | [Stream API](https://www.runoob.com/java/java9-stream-api-improvements.html) |
|  9   | [try-with-resources](https://www.runoob.com/java/java9-try-with-resources-improvement.html) |
|  10  | [@Deprecated](https://www.runoob.com/java/java9-enhanced-deprecated-annotation.html) |
|  11  | [内部类的钻石操作符(Diamond Operator)](https://www.runoob.com/java/java9-inner-class-diamond-operator.html) |
|  12  | [Optional 类](https://www.runoob.com/java/java9-optional-class-improvements.html) |
|  13  | [多分辨率图像 API](https://www.runoob.com/java/java9-multiresolution-image_api.html) |
|  14  | [CompletableFuture API](https://www.runoob.com/java/java9-completablefuture-api-improvements.html) |

# 模块化系统

[JDK9模块化知识和规则入门 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/347112501#:~:text=）更简单的说法是：模块%3D代码%2B数据。,Java9模块系统的主要目标是支持Java中的模块化编程。)

模块是代码、数据和资源的集合。它是一组相关的包和类型（类、抽象类、接口等），包含代码、数据文件和一些静态资源。

![img](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206140939265.jpeg)

Java9模块系统的主要目标是支持Java中的模块化编程。

在Java8和更早的应用程序中，顶级组件是包 **package** 。它将一组相关类型放入一个组中。它还包含一组资源。

java9应用程序与java8没有太大区别；它引入了一个新组件 **module** ，用于将一组相关的包放入一个组中。

同时还介绍了另一个新组件：模块描述符module-info.java

注：Java8应用程序将包作为顶级组件，Java9应用程序将模块作为顶级组件。

# 集合工厂方法

List，Set 和 Map 接口中，新的静态工厂方法可以创建这些集合的不可变实例。

理解：

1、源码理解。

```java
static <E> List<E> of(E e1, E e2, E e3);
static <E> Set<E> of(E e1, E e2, E e3);
static <K,V> Map<K,V> of(K k1, V v1, K k2, V v2, K k3, V v3);
static <K,V> Map<K,V> ofEntries(Map.Entry<? extends K,? extends V>... entries)
//实例
Set<String> set = Set.of("A", "B", "C");      
System.out.println(set);

List<String> list = List.of("A", "B", "C");
System.out.println(list);

Map<String, String> map = Map.of("A","Apple","B","Boy","C","Cat");
System.out.println(map);

Map<String, String> map1 = Map.ofEntries (
new AbstractMap.SimpleEntry<>("A","Apple"),
new AbstractMap.SimpleEntry<>("B","Boy"),
new AbstractMap.SimpleEntry<>("C","Cat"));
System.out.println(map1);

```

2、方法理解:

1. 不可变就是不能改变List的add、remove方法都不能使用；
2. 否则会报错，java.lang.UnsupportedOperationException(不支持请求操作)——抛出的RuntimeException的子类

# 私有接口方法

在接口中使用private私有方法。我们可以使用 private 访问修饰符在接口中编写私有方法。

理解：

1、在 Java 9 中，一个接口中能定义如下几种变量/方法：

- 常量（8）
- 抽象方法（8）
- 默认方法（8）
- 静态方法（8）
- 私有方法
- 私有静态方法

2、接口中的私有方法作用：**可以被默认方法调用（这就体现了私有方法的存在意义）**

> 用来解决两个默认方法之间重复代码的问题。
>
> 但是这个共有方法不应该让实现类使用，应该私有化。

# 补充Optional类

Optional 类在 Java 8 中引入，Optional 类的引入很好的解决空指针异常。。在 java 9 中, 添加了三个方法来改进它的功能：

- stream()
- ifPresentOrElse()
- or()

## stream() 方法

stream 方法的作用就是将 Optional 转为一个 Stream，如果该 Optional 中包含值，那么就返回包含这个值的 Stream，否则返回一个空的 Stream（Stream.empty()）。

```java
package com.java_2_advanced.JDK9Test;
import java.util.Arrays;
import java.util.List;
import java.util.Optional;
import java.util.stream.Collectors;
import java.util.stream.Stream;

public class OptionTest {
    public static void main(String[] args) {
        List<Optional<String>> list = Arrays.asList (
                Optional.empty(),
                Optional.of("A"),
                Optional.empty(),
                Optional.of("B"));
        list.forEach(s-> System.out.println(s));

        //如果 optional 为非空，则过滤列表以打印非空值，获取流中的值，否则返回空
        List<String> filteredList = list.stream()
                .flatMap(o -> o.isPresent() ? Stream.of(o.get()) : Stream.empty())
                .collect(Collectors.toList());

        //如果数据存在或不存在，可选::stream 方法将返回一个元素或零元素的流
        List<String> filteredListJava9 = list.stream()
                .flatMap(Optional::stream)
                .collect(Collectors.toList());

        System.out.println(filteredList);
        System.out.println(filteredListJava9);
    }
}
```

## ifPresentOrElse() 方法

**语法**

```java
public void ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction)
```

ifPresentOrElse 方法的改进就是有了 else，接受两个参数 Consumer 和 Runnable。

> ifPresentOrElse 方法的用途是，如果一个 Optional 包含值，则对其包含的值调用函数 action，即 action.accept(value)，这与 ifPresent 一致；
>
> 与 ifPresent 方法的区别在于，ifPresentOrElse 还有第二个参数 emptyAction —— 如果 Optional 不包含值，那么 ifPresentOrElse 便会调用 emptyAction，即 emptyAction.run()。

```java
import java.util.Optional;
 
public class Tester {
   public static void main(String[] args) {
      Optional<Integer> optional = Optional.of(1);
 
      optional.ifPresentOrElse( x -> System.out.println("Value: " + x),() -> 
         System.out.println("Not Present."));
 
      optional = Optional.empty();
 
      optional.ifPresentOrElse( x -> System.out.println("Value: " + x),() -> 
         System.out.println("Not Present."));
   }  
}
```

Or方法

**语法**

```java
public Optional<T> or(Supplier<? extends Optional<? extends T>> supplier)
```

如果值存在，返回 Optional 指定的值，否则返回一个预设的值。

```java
 Optional<String> optional1 = Optional.of("djx");
        Supplier<Optional<String>> supplierString = () -> Optional.of("不存在");

        optional1 = optional1.or(supplierString);
        optional1.ifPresent(x -> System.out.println("Value: " + x));
        
        optional1 = Optional.empty();
        optional1 = optional1.or(supplierString);
        optional1.ifPresent(x -> System.out.println("Value: " + x));
//Value: djx
//Value: 不存在
```

# 新的进程API

在 Java 9 之前，Process API 仍然缺乏对使用本地进程的基本支持，例如获取进程的 PID 和所有者，进程的开始时间，进程使用了多少 CPU 时间，多少本地进程正在运行等。

Java 9 向 Process API 添加了一个名为 ProcessHandle 的接口来增强 java.lang.Process 类。

ProcessHandle 接口的实例标识一个本地进程，它允许查询进程状态并管理进程。

ProcessHandle 嵌套接口 Info 来让开发者逃离时常因为要获取一个本地进程的 PID 而不得不使用本地代码的窘境。

我们不能在接口中提供方法实现。如果我们要提供抽象方法和非抽象方法（方法与实现）的组合，那么我们就得使用抽象类。

ProcessHandle 接口中声明的 onExit() 方法可用于在某个进程终止时触发某些操作。

```java
public static void main(String[] args) throws IOException {
      ProcessBuilder pb = new ProcessBuilder("notepad.exe");
      String np = "Not Present";
      Process p = pb.start();
      ProcessHandle.Info info = p.info();
      System.out.printf("Process ID : %s%n", p.pid());
      System.out.printf("Command name : %s%n", info.command().orElse(np));
      System.out.printf("Command line : %s%n", info.commandLine().orElse(np));
 
      System.out.printf("Start time: %s%n",
         info.startInstant().map(i -> i.atZone(ZoneId.systemDefault())
         .toLocalDateTime().toString()).orElse(np));
 
      System.out.printf("Arguments : %s%n",
         info.arguments().map(a -> Stream.of(a).collect(
         Collectors.joining(" "))).orElse(np));
 
      System.out.printf("User : %s%n", info.user().orElse(np));
   } 
/*
Process ID : 5800
Command name : C:\Windows\System32\notepad.exe
Command line : Not Present
Start time: 2022-06-04T21:35:03.626
Arguments : Not Present
User: administrator
*/
```

