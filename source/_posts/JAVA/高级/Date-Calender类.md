---
title: date、calender类
date: 2022-06-16 10:59:37
categories:
- JAVA
- 高级
---

[Java时间日期的处理：Java Date类、Calendar类详解 ](http://c.biancheng.net/view/876.html)

[Java日期格式化（DateFormat类和SimpleDateFormat类）](http://c.biancheng.net/view/878.html)

[java-将字符串和毫秒值转化为日期格式的几种方法](https://blog.csdn.net/qq_32475739/article/details/64934903)

[Java8新特性之Instant详解_corleone_](https://blog.csdn.net/corleone_4ever/article/details/104403042)



**重点理解记忆：**

**注：Date 类主要封装了系统的日期和时间的信息，Calendar 类则会根据系统的日历来解释 Date 对象**

# Date类：（**util**包）  

 Date 对象表示时间的默认顺序是 星期、月、日、小时、分、秒、年。

## 构造方法：

a、空参构造方法：得到当前系统时间的Date对象。

b、long类型参数的构造函数（long date）：参数系统毫秒时间，得到系统毫秒时间的Date对象——用处：时间毫秒转化为Date对象。

注：**从 GMT 时间（格林尼治时间）1970 年 1 月 1 日 0 时 0 分 0 秒开始经过参数 date 指定的毫秒数**

**System.currentTimeMillis()：返回当前系统时间，**

## 常用方法

![image-20220616110228449](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206161102587.png)

| boolean after(Date when)        |                 判断此日期是否在指定日期之后                 |
| :------------------------------ | :----------------------------------------------------------: |
| boolean before(Date when)       |                 判断此日期是否在指定日期之前                 |
| int compareTo(Date anotherDate) |        比较两个日期的顺序 。1前者大；0相等；-1后者大         |
| boolean equals(Object obj)      |            比较两个日期的相等性（Date对象的比较）            |
| long getTime()                  | 返回自 1970 年 1 月 1 日 00:00:00 GMT 以来，此 Date  对象表示的**毫秒数**    **把日期对象转换成对应的时间毫秒值。** |
| String toString()               | 把此 Date 对象转换为以下形式的 String: dow mon dd hh:mm:ss zzz yyyy。  其中 date 是一周中的某一天(Sun、Mon、Tue、Wed、Thu、Fri 及 Sat) |
| String toLocaleString()         |       当前Date对象转化为（ 年 月 日 时：分：秒）格式。       |

(毫秒-》1000-秒-》60-分-》60-时-》24-天-》31/30/29/28-月)

![image-20220616110545271](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206161105313.png)

## 格式化日期类

（DateFormat类和SimpleDateFormat类）的格式化日期类。

a、DateFormat 是日期/时间格式化子类的抽象类；

   SimpleDateFormat日期/时间格式化子类，允许进行格式化（format：也就是日期→文本String）、解析（parse：文本String→日期）和标准化日期。

b、使用 DateFomat 类可以对日期进行不同风格的格式化。

在创建 DateFormat 对象时不能使用 new 关键字，而应该使用 DateFormat 类中的静态方法 getDateInstance()，示例代码如下：

```java
DateFormat df = DateFormat.getDatelnstance();

//常用当时往往都是：

DateFormat dateFormat=new SimpleDateFormat("yyyy-MM-dd");

//new格式化日期类的子类对象。
```

 

eg：

```java
Date now = new Date(); 
// 创建一个Date对象，获取当前时间

SimpleDateFormat f = new SimpleDateFormat("今天是 " + "yyyy 年 MM 月 dd 日 E     HH 点 mm 分 ss 秒");
// 指定格式化格式

System.out.println(f.format(now)); 
// 将当前时间袼式化为指定的格式Date-->String

//DateFormat 类格式化日期/时间并不能满足要求，那么就需要使用 DateFormat 类的子类——SimpleDateFormat
```



a、SimpleDateFormat 类主要有如下 3 种构造方法。 

```java
SimpleDateFormat() //用默认的格式和默认的语言环境构造

SimpleDateFormat(String pattern) //用指定的格式和默认的语言环境构造 。

SimpleDateFormat(String pattern,Locale locale) //用指定的格式和指定的语言环境构造 （Locale.CHINA，Locale.US）
```

 

**指定格式规则**：没有“Y”

| y    | 年份。一般用 yy 表示两位年份，yyyy 表示 4  位年份            | 使用 yy 表示的年份，如 11；  使用 yyyy 表示的年份，如 2011   |
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| M    | 月份。一般用 MM 表示月份，如果使用 MMM，则会  根据语言环境显示不同语言的月份 | 使用 MM 表示的月份，如 05；  使用 MMM 表示月份，在 Locale.CHINA  语言环境下，如“十月”；  在 Locale.US语言环境下，如 Oct |
| d    | 月份中的天数。一般用 dd 表示天数                             | 使用 dd 表示的天数，如 10                                    |
| D    | 年份中的天数。表示当天是当年的第几天， 用 D 表示             | 使用 D 表示的年份中的天数，如 295                            |
| E    | 星期几。用 E 表示，会根据语言环境的不同，  显示不同语言的星期几 | 使用 E 表示星期几，在 Locale.CHINA 语  言环境下，如“星期四”；在 Locale.US 语  言环境下，如 Thu |
| H    | 一天中的小时数（0~23)。一般用 HH 表示小时数                  | 使用 HH 表示的小时数，如 18                                  |
| h    | 一天中的小时数（1~12)。一般使用 hh 表示小时数                | 使用 hh 表示的小时数，如 10 (注意 10 有  可能是 10 点，也可能是 22 点） |
| m    | 分钟数。一般使用 mm 表示分钟数                               | 使用 mm 表示的分钟数，如 29                                  |
| s    | 秒数。一般使用 ss 表示秒数                                   | 使用 ss 表示的秒数，如 38                                    |
| S    | 毫秒数。一般使用 SSS 表示毫秒数                              | 使用 SSS 表示的毫秒数，如 156                                |
| a    | Am、pm标记上午还是下午                                       | 往往使用在1-12时判断上午、下午                               |





#  Calendar类：（util包）

是抽象类也不能new

1、由于语言敏感性，Calendar类在创建对象时并非直接创建，而是通过静态方法（getInstance方法）创建，返回子类对象

eg：

```java
Calendar cal = Calendar.getInstance();

Calendar calendar=new GregorianCalendar();
```

![image-20220616170026015](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206161700071.png)

常用方法：

| public int get(int field)                  |                    返回给定日历字段的值。                    |
| :----------------------------------------- | :----------------------------------------------------------: |
| void set(int field, int value)             |                将给定的日历字段设置为给定值。                |
| abstract void add(int field, int amount)   |  根据日历的规则，为给定的日历字段 添加或减去指定的时间量。   |
| Date  getTime()   void  setTime(Date date) | 返回一个表示此Calendar时间值（从历元到现在的毫秒偏移量）的  Date对象。      存入一个Date类型就变成Calendar类型 |
| void clear()                               |                        清空日期时间值                        |
| int compareTo(Calendar anotherCalendar)    | 比较两个 Calendar 对象表示的时间值（从格林威治时间  1970 年 01 月 01 日  00 时 00 分 00 秒至现在的毫秒偏移量），大则返回  1，小则返回 -1，相等返回 0 |
| void  setTimeInMillis(long millis)         |        用给定的 long  值设置此 Calendar 的当前时间值         |
| boolean after/before(Object when)          | 判断此 Calendar 表示的时间是否在指定时间 when 之后，并返回判断结果 |

 

3、Calendar 对象可以调用 set() 方法将日历指定到任何一个时间，当参数 year 取负数时表示公元前。

Calendar 对象调用 get() 方法可以获取有关年、月、日等时间信息，参数 field 的有效值由 Calendar 静态常量指定。

 

注：

4、Calendar 类中定义了许多常量，分别表示不同的意义。

- Calendar.YEAR：年份。
- Calendar.MONTH：月份。（0-11）
- Calendar.DATE：日期。
- Calendar.DAY_OF_MONTH：日期，和上面的字段意义完全相同（几号）。
- Calendar.HOUR：12小时制的小时。
- Calendar.HOUR_OF_DAY：24 小时制的小时。
- Calendar.MINUTE：分钟。
- Calendar.SECOND：秒。
- Calendar.DAY_OF_WEEK：星期几（外国周日为第一天，-1就可以使用）。

 

5、get/set/add实例 getTime方法

```java
//**创建**Calendar对象*

Calendarcal=Calendar.*getInstance*();

intyear=cal.get(Calendar.*YEAR*);

intmonth=cal.get(Calendar.*MONTH*)+1;

intdayOfMonth=cal.get(Calendar.*DAY_OF_MONTH*);

cal.set(Calendar.*YEAR*,2020);

cal.add(Calendar.*DAY_OF_MONTH*,2);*//**加**2天*

cal.add(Calendar.*YEAR*,-3);*//**减**3年*

Calendarcal2=Calendar.*getInstance*();

Datedate2=cal2.getTime();
```

 

注：Calendar类的getTime方法返回值时Date类

 

 

 

### 总结：

**Date---》String（SimpleDateFormat类的format方法）**

**String---》Date（SimpleDateFormat类的parse方法）**

**Date---》毫秒时间（getTime方法）**

**毫秒时间---》Date（Date类的long型参数的构造方法）**

**Calendar---》Date（getTime方法）**

**Date---》Calendar（setTime方法）**





# LocalDateTime 、ZoneDateTime 、Instant理解

见JDK1.8新特性中总结。