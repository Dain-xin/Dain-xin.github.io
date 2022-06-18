---
title: 枚举Enum
date: 2022-06-18 17:52:37
categories:
- JAVA
- 高级
---

1、类的对象只有有限个，确定的。eg:星期、季节、运行状态….
2、当需要定义一组常量时，强烈建议使用枚举类
3、JDK1.5以前是自定义枚举类，JDK1.5后新增的 enum 关键字用于定义枚举类。
4、若枚举只有一个对象, 则可以作为一种单例模式的实现方式。
	单例模式：一个类只有一个实例，且该类能自行创建这个实例的一种模式。
5、对象属性：private final ；

类的内部创建枚举类的实例：public static final

```java

private final String SEASONNAME;
//季节的名称 
private final String SEASONDESC;//季节的描述
	private Season(String seasonName,String seasonDesc){  
        this.SEASONNAME = seasonName;
		this.SEASONDESC = seasonDesc;
	}
	public static final Season SPRING = new Season("春天", "春暖花开");  

```

6、Enum关键字：（lang包.Enum）默认继承Enum类。
	实例默认：public static final；构造方法默认：private；属性还是private final。
7、常用方法：
		○ values()方法：返回枚举类型的对象数组。该方法可以很方便地遍历所有的
	枚举值。
		○ valueOf(String str)：可以把一个字符串转为对应的枚举类对象。要求字符 串必须是枚举类对象的“名字”。如不是，会有运行时异常：  IllegalArgumentException。
		○ toString()：返回当前枚举类对象常量的名称
		○ 从Object类继承来的方法，clone、hashcode、toString、equals…
8、实现多个接口，但是自身默认继承Enum类就不能在继承。