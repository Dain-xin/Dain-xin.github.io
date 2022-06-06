---
title: JDK1.8特性
date: 2022-06-03 22:26:52
categories:
- JAVA
- 基础
---



### 一、接口的默认方法

Java 8允许我们给接口添加一个非抽象的方法实现，只需要使用 default关键字即可，这个特征又叫做扩展方法，

代码如下:

```java
interface Formula { 
	double calculate(int a);

	default double sqrt(int a) { 
	return Math.sqrt(a); 
	} 
}
```

>  Formula接口在拥有calculate方法之外同时还定义了sqrt方法，实现了Formula接口的子类只需要实现一个calculate方法，默认方法sqrt将在子类上可以直接使用。

代码如下:

```java
Formula formula = new Formula() { 
	@Override 
    public double calculate(int a) { 					return sqrt(a * 100); 
	} 
};

formula.calculate(100); // 100.0 
formula.sqrt(16); // 4.0
```

文中的formula被实现为一个匿名类的实例，该代码非常容易理解，6行代码实现了计算 sqrt(a * 100)。在下一节中，我们将会看到实现单方法接口的更简单的做法。

> 理解：默认方法可以实现接口不需要些改动的方法，同时子类还可以重写，没有限制。
>
> JDK9出来的私有方法的使用可以在默认方法中进行，避免代码冗余，降低耦合。
>
> 同时默认方法可以是静态的，调用私有静态方法

---

### 二、Lambda 表达式

首先看看在老版本的Java中是如何排列字符串的：代码如下:

```java
List<String> names = Arrays.asList("peterF", "anna", "mike", "xenia");
Collections.sort(names, new Comparator<String>() { 
    @Override 
    public int compare(String a, String b) 
    { 
        return b.compareTo(a); 
    } 
});
```

只需要给静态方法 Collections.sort 传入一个List对象以及一个比较器来按指定顺序排列。通常做法都是创建一个匿名的比较器对象然后将其传递给sort方法。

在Java 8 中你就没必要使用这种传统的匿名对象的方式了，Java 8提供了更简洁的语法，lambda表达式：

代码如下:

```
Collections.sort(names, (String a, String b) -> { return b.compareTo(a); });
```

看到了吧，代码变得更段且更具有可读性，但是实际上还可以写得更短：

代码如下:

```java
Collections.sort(names, (String a, String b) -> b.compareTo(a));
```

对于函数体只有一行代码的，你可以去掉大括号{}以及return关键字，但是你还可以写得更短点：

代码如下:

```java
Collections.sort(names, (a, b) -> b.compareTo(a));
```

Java编译器可以自动推导出参数类型，所以你可以不用再写一次类型。接下来我们看看lambda表达式还能作出什么更方便的东西来：

- Lambda 表达式，也可称为闭包，它是推动 Java 8 发布的最重要新特性。Lambda     允许把函数作为一个方法的参数（函数作为参数传递进方法中）。使用 Lambda 表达式可以使代码变的更加简洁紧凑。
- 面向对象的思想:     做一件事情,找一个能解决这个事情的对象,**调用对象的方法,完成事情.**   函数式编程思想:  只要能获取到结果,谁去做的,怎么做的都不重要,**重视的是结果,不重视过程**。
- 这是一种比匿名内部类更为简单的一种写法，只允许拥有一种抽象方法，或者只剩一种抽象方法没有实现（Compartor接口里面有两种抽象方法，compareTo 和 equals 方法）但是Object类都有equals方法默认实现了该抽象方法。

使用Lambda必须具有上下文推断。也就是方法的参数或局部变量类型必须为Lambda对应的接口类型，才能使用Lambda作为该接口的实例。

1. 写法理解：

**(参数类型 参数名称) ‐> { 代码语句 }** 

Lambda表达式的标准格式为：格式说明：

1. 小括号内的语法与传统方法参数列表一致：无参数则留空；多个参数则用逗号分隔。
2. ->是新引入的语法格式，代表指向动作。

eg1：

```java
// 1. 不需要参数,返回值为 5 
 () -> 5   
 // 2. 接收一个参数(数字类型),返回其2倍的值 
 x -> 2 * x   
 // 3. 接受2个参数(数字),并返回他们的差值 
 (x, y) -> x – y   
 // 4. 接收2个int型整数,返回他们的和 
 (int x, int y) -> x + y   
 // 5. 接受一个 string 对象,并在控制台打印,不返回任何值(看起来像是返回void) 
 (String s) -> System.out.print(s) 
```

eg2:**简单实现四大法则：**

```java
private static void lambda表达式(){
sf_ab(130,120,"+",(Integer a,Integer b,String h)->a+b);
sf_ab(130,120,"-",(Integer a,Integer b,String h)->a-b);
sf_ab(130,120,"*",(Integer a,Integerb,Stringh)->a*b);
sf_ab(130,120,"/",(Integer a,Integer b,String h)->a/b);
}

private static void sf_ab(int a,int b,String h,算法<Integer>t){
	Integer sf=t.sf(a,b,h);
	System.out.println(a+h+b+"结果："+sf);
	}
}

interface 算法<T>{
	Tsf(T a,T b,String h);
}
```

 

---

### 三、函数式接口

1、函数式接口(Functional Interface)就是一个有且仅有一个抽象方法，但是可以有多个非抽象方法的接口。函数式接口可以被隐式转换为 lambda 表达式。

代码如下:

```java
@FunctionalInterface 
interface Converter<F, T> { 
    T convert(F from); 
} 
Converter<String, Integer> converter = (from) -> Integer.valueOf(from); 
Integer converted = converter.convert("123");
System.out.println(converted); // 123
```

2、需要注意如果@FunctionalInterface如果没有指定，上面的代码也是对的。函数式接口可以对现有的函数友好地支持 lambda。

JDK 1.8 之前已有的函数式接口:

- java.lang.Runnable
- java.util.concurrent.Callable
- java.util.Comparator
- java.io.FileFilter

JDK 1.8 的java.util.function 包的接口：

#### **Supplier接口：**

抽象方法：

T get（） ：用来获取一个泛型参数指定类型的对象数据。

由于这是一个函数式接口，这也就意味着对应的Lambda表达式需要“对外提供”一个符合泛型类型的对象数据。

#### **Consumer接口：**

抽象方法：

void accept （T t）：处理一个指定泛型的数据。

默认方法：

Consumer andThen (Consumer after)：

如果一个方法的参数和返回值全都是 Consumer 类型，那么就可以实现效果：消费数据的时候，首先做一个操作， 然后再做一个操作，实现组合。而这个方法就是 Consumer 接口中的default方法 andThen 。

**实例：**one.andThen(two).accept(info);

**源码：**

![image-20220607000545520](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206070005623.png)

#### **Predicate接口：**

对某种类型的数据进行判断，从而得到一个boolean值结果；

**抽象方法：**

boolean test(T t)；

**默认方法：**

条件判断，就会存在与、或、非三种常见的逻辑关系

**实例：**

| one.and(two).test("Helloworld");       | 与   |
| -------------------------------------- | ---- |
| one.or(two).test("Helloworld");        | 或   |
| predicate.negate().test("HelloWorld"); | 非   |

![image-20220607000725009](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206070007254.png)

#### Function接口：

**抽象方法：**

R apply(T t)：

类型T的参数获取类型R的结果

 ![image-20220607000813629](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206070008806.png)

3、当接口有两个抽象方法的时候，就不在是函数式接口了，使用@FunctionalInterface标注编译时会报错

![image-20220606235850396](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206062358453.png)

这里有3个抽象方法，不报错？

​    我们知道**toString和equals方法是Object的方法，Java基础告诉我们，Object是所有类的默认父类，也就是说任何对象都会包含Object里面的方法，即使是函数式接口的实现，也会有Object的默认方法，**

**所以：**重写Object中的方法，不会计入接口方法中，除了final不能重写的，Object中所能重写的方法，写到接口中，不会影响函数式接口的特性

4、JAVA8允许有默认方法，所以默认方法不影响函数式接口的规则。

5、

**理解：**

- @FunctionalInterface注解标注一个函数式接口，不能标注类，方法，枚举，属性这些.
- 如果接口被标注了@FunctionalInterface，这个类就必须符合函数式接口的规范.
- 即使一个接口没有标注@FunctionalInterface，如果这个接口满足函数式接口规则，依旧被当作函数式接口。

### 四、方法与构造函数引用

前一节中的代码还可以通过静态方法引用来表示：

代码如下:

Converter<String, Integer> converter = Integer::valueOf; Integer converted = converter.convert("123"); System.out.println(converted); // 123

Java 8 允许你使用 :: 关键字来传递方法或者构造函数引用，上面的代码展示了如何引用一个静态方法，我们也可以引用一个对象的方法：

代码如下:

converter = something::startsWith; String converted = converter.convert("Java"); System.out.println(converted); // "J"

接下来看看构造函数是如何使用::关键字来引用的，首先我们定义一个包含多个构造函数的简单类：

代码如下:

class Person { String firstName; String lastName;

Person() {}

Person(String firstName, String lastName) { this.firstName = firstName; this.lastName = lastName; } }

接下来我们指定一个用来创建Person对象的对象工厂接口：

代码如下:

interface PersonFactory<P extends Person> { P create(String firstName, String lastName); }

这里我们使用构造函数引用来将他们关联起来，而不是实现一个完整的工厂：

代码如下:

PersonFactory<Person> personFactory = Person::new; Person person = personFactory.create("Peter", "Parker");

我们只需要使用 Person::new 来获取Person类构造函数的引用，Java编译器会自动根据PersonFactory.create方法的签名来选择合适的构造函数。

### 五、Lambda 作用域

在lambda表达式中访问外层作用域和老版本的匿名对象中的方式很相似。你可以直接访问标记了final的外层局部变量，或者实例的字段以及静态变量。

### 六、访问局部变量

我们可以直接在lambda表达式中访问外层的局部变量：

代码如下:

```java
final int num = 1; 
Converter<Integer, String> stringConverter = (from) -> String.valueOf(from + num);
stringConverter.convert(2); // 3
```

但是和匿名对象不同的是，这里的变量num可以不用声明为final，该代码同样正确：

代码如下:

```java
int num = 1; 
Converter<Integer, String> stringConverter = (from) -> String.valueOf(from + num);
stringConverter.convert(2); // 3
```

不过这里的num必须不可被后面的代码修改（即隐性的具有final的语义），例如下面的就无法编译：

代码如下:

```java
int num = 1; 
Converter<Integer, String> stringConverter = (from) -> String.valueOf(from + num); 
num = 3;
```

在lambda表达式中试图修改num同样是不允许的。

### 七、访问对象字段与静态变量

和本地变量不同的是，**lambda内部对于实例的字段以及静态变量是即可读又可写**。

该行为和匿名对象是一致的：

代码如下:

```java
class Lambda4 { 
    static int outerStaticNum; 
    int outerNum;

void testScopes() 
{ 
    Converter<Integer, String> stringConverter1 = (from) -> { outerNum = 23; return String.valueOf(from);};

Converter<Integer, String> stringConverter2 = (from) -> {  outerStaticNum = 72;  return String.valueOf(from);  }; } }
```



### 八、访问接口的默认方法

JDK 1.8 API包含了很多内建的函数式接口，在老Java中常用到的比如Comparator或者Runnable接口，这些接口都增加了@FunctionalInterface注解以便能用在lambda上。

 Java 8 API同样还提供了很多全新的函数式接口来让工作更加方便，有一些接口是来自Google Guava库里的

**Predicate**接口

Predicate 接口只有一个参数，返回boolean类型。该接口包含多种默认方法来将Predicate组合成其他复杂的逻辑（比如：与，或，非）：

代码如下:

Predicate<String> predicate = (s) -> s.length() > 0;

predicate.test("foo"); // true predicate.negate().test("foo"); // false

Predicate<Boolean> nonNull = Objects::nonNull; Predicate<Boolean> isNull = Objects::isNull;

Predicate<String> isEmpty = String::isEmpty; Predicate<String> isNotEmpty = isEmpty.negate();

**Function** **接口**

Function 接口有一个参数并且返回一个结果，并附带了一些可以和其他函数组合的默认方法（compose, andThen）：

代码如下:

Function<String, Integer> toInteger = Integer::valueOf; Function<String, String> backToString = toInteger.andThen(String::valueOf);

backToString.apply("123"); // "123"

**Supplier** **接口** Supplier 接口返回一个任意范型的值，和Function接口不同的是该接口没有任何参数

代码如下:

Supplier<Person> personSupplier = Person::new; personSupplier.get(); // new Person

**Consumer** **接口** Consumer 接口表示执行在单个参数上的操作。

代码如下:

Consumer<Person> greeter = (p) -> System.out.println("Hello, " + p.firstName); greeter.accept(new Person("Luke", "Skywalker"));

**Comparator** **接口** Comparator 是老Java中的经典接口， Java 8在此之上添加了多种默认方法：

代码如下:

Comparator<Person> comparator = (p1, p2) -> p1.firstName.compareTo(p2.firstName);

Person p1 = new Person("John", "Doe"); Person p2 = new Person("Alice", "Wonderland");

comparator.compare(p1, p2); // > 0 comparator.reversed().compare(p1, p2); // < 0

**Optional** **接口**

Optional 不是函数是接口，这是个用来防止NullPointerException异常的辅助类型，这是下一届中将要用到的重要概念，现在先简单的看看这个接口能干什么：

Optional 被定义为一个简单的容器，其值可能是null或者不是null。在Java 8之前一般某个函数应该返回非空对象但是偶尔却可能返回了null，而在Java 8中，不推荐你返回null而是返回Optional。

代码如下:

Optional<String> optional = Optional.of("bam");

optional.isPresent(); // true optional.get(); // "bam" optional.orElse("fallback"); // "bam"

optional.ifPresent((s) -> System.out.println(s.charAt(0))); // "b"

**Stream** **接口**

java.util.Stream 表示能应用在一组元素上一次执行的操作序列。Stream 操作分为中间操作或者最终操作两种，最终操作返回一特定类型的计算结果，而中间操作返回Stream本身，这样你就可以将多个操作依次串起来。Stream 的创建需要指定一个数据源，比如 java.util.Collection的子类，List或者Set， Map不支持。Stream的操作可以串行执行或者并行执行。

首先看看Stream是怎么用，首先创建实例代码的用到的数据List：

代码如下:

List<String> stringCollection = new ArrayList<>(); stringCollection.add("ddd2"); stringCollection.add("aaa2"); stringCollection.add("bbb1"); stringCollection.add("aaa1"); stringCollection.add("bbb3"); stringCollection.add("ccc"); stringCollection.add("bbb2"); stringCollection.add("ddd1");

Java 8扩展了集合类，可以通过 Collection.stream() 或者 Collection.parallelStream() 来创建一个Stream。下面几节将详细解释常用的Stream操作：

**Filter** **过滤**

过滤通过一个predicate接口来过滤并只保留符合条件的元素，该操作属于中间操作，所以我们可以在过滤后的结果来应用其他Stream操作（比如forEach）。forEach需要一个函数来对过滤后的元素依次执行。forEach是一个最终操作，所以我们不能在forEach之后来执行其他Stream操作。

代码如下:

stringCollection .stream() .filter((s) -> s.startsWith("a")) .forEach(System.out::println);

// "aaa2", "aaa1"

**Sort** **排序**

排序是一个中间操作，返回的是排序好后的Stream。如果你不指定一个自定义的Comparator则会使用默认排序。

代码如下:

stringCollection .stream() .sorted() .filter((s) -> s.startsWith("a")) .forEach(System.out::println);

// "aaa1", "aaa2"

需要注意的是，排序只创建了一个排列好后的Stream，而不会影响原有的数据源，排序之后原数据stringCollection是不会被修改的：

代码如下:

System.out.println(stringCollection); // ddd2, aaa2, bbb1, aaa1, bbb3, ccc, bbb2, ddd1

**Map** **映射** 中间操作map会将元素根据指定的Function接口来依次将元素转成另外的对象，下面的示例展示了将字符串转换为大写字符串。你也可以通过map来讲对象转换成其他类型，map返回的Stream类型是根据你map传递进去的函数的返回值决定的。

代码如下:

stringCollection .stream() .map(String::toUpperCase) .sorted((a, b) -> b.compareTo(a)) .forEach(System.out::println);

// "DDD2", "DDD1", "CCC", "BBB3", "BBB2", "AAA2", "AAA1"

**Match** **匹配**

Stream提供了多种匹配操作，允许检测指定的Predicate是否匹配整个Stream。所有的匹配操作都是最终操作，并返回一个boolean类型的值。

代码如下:

boolean anyStartsWithA = stringCollection .stream() .anyMatch((s) -> s.startsWith("a"));

System.out.println(anyStartsWithA); // true

boolean allStartsWithA = stringCollection .stream() .allMatch((s) -> s.startsWith("a"));

System.out.println(allStartsWithA); // false

boolean noneStartsWithZ = stringCollection .stream() .noneMatch((s) -> s.startsWith("z"));

System.out.println(noneStartsWithZ); // true

**Count** **计数** 计数是一个最终操作，返回Stream中元素的个数，返回值类型是long。

代码如下:

long startsWithB = stringCollection .stream() .filter((s) -> s.startsWith("b")) .count();

System.out.println(startsWithB); // 3

**Reduce** **规约**

这是一个最终操作，允许通过指定的函数来讲stream中的多个元素规约为一个元素，规越后的结果是通过Optional接口表示的：

代码如下:

Optional<String> reduced = stringCollection .stream() .sorted() .reduce((s1, s2) -> s1 + "#" + s2);

reduced.ifPresent(System.out::println); // "aaa1#aaa2#bbb1#bbb2#bbb3#ccc#ddd1#ddd2"

**并行****Streams**

前面提到过Stream有串行和并行两种，串行Stream上的操作是在一个线程中依次完成，而并行Stream则是在多个线程上同时执行。

下面的例子展示了是如何通过并行Stream来提升性能：

首先我们创建一个没有重复元素的大表：

代码如下:

int max = 1000000; List<String> values = new ArrayList<>(max); for (int i = 0; i < max; i++) { UUID uuid = UUID.randomUUID(); values.add(uuid.toString()); }

然后我们计算一下排序这个Stream要耗时多久， 串行排序：

代码如下:

long t0 = System.nanoTime();

long count = values.stream().sorted().count(); System.out.println(count);

long t1 = System.nanoTime();

long millis = TimeUnit.NANOSECONDS.toMillis(t1 - t0); System.out.println(String.format("sequential sort took: %d ms", millis));

// 串行耗时: 899 ms 并行排序：

代码如下:

long t0 = System.nanoTime();

long count = values.parallelStream().sorted().count(); System.out.println(count);

long t1 = System.nanoTime();

long millis = TimeUnit.NANOSECONDS.toMillis(t1 - t0); System.out.println(String.format("parallel sort took: %d ms", millis));

// 并行排序耗时: 472 ms 上面两个代码几乎是一样的，但是并行版的快了50%之多，唯一需要做的改动就是将stream()改为parallelStream()。

**Map**

前面提到过，Map类型不支持stream，不过Map提供了一些新的有用的方法来处理一些日常任务。

代码如下:

Map<Integer, String> map = new HashMap<>();

for (int i = 0; i < 10; i++) { map.putIfAbsent(i, "val" + i); }

map.forEach((id, val) -> System.out.println(val)); 以上代码很容易理解， putIfAbsent 不需要我们做额外的存在性检查，而forEach则接收一个Consumer接口来对map里的每一个键值对进行操作。

下面的例子展示了map上的其他有用的函数：

代码如下:

map.computeIfPresent(3, (num, val) -> val + num); map.get(3); // val33

map.computeIfPresent(9, (num, val) -> null); map.containsKey(9); // false

map.computeIfAbsent(23, num -> "val" + num); map.containsKey(23); // true

map.computeIfAbsent(3, num -> "bam"); map.get(3); // val33

接下来展示如何在Map里删除一个键值全都匹配的项：

代码如下:

map.remove(3, "val3"); map.get(3); // val33

map.remove(3, "val33"); map.get(3); // null

另外一个有用的方法：

代码如下:

map.getOrDefault(42, "not found"); // not found

对Map的元素做合并也变得很容易了：

代码如下:

map.merge(9, "val9", (value, newValue) -> value.concat(newValue)); map.get(9); // val9

map.merge(9, "concat", (value, newValue) -> value.concat(newValue)); map.get(9); // val9concat

Merge做的事情是如果键名不存在则插入，否则则对原键对应的值做合并操作并重新插入到map中。

### 九、Date API

Java 8 在包java.time下包含了一组全新的时间日期API。

新的时间API  **（LocalDate/LocalTime 和 LocalDateTime 类）**[LocalDateTime](https://www.liaoxuefeng.com/wiki/1252599548343744/1303871087444002)

#### **Clock** **时钟**

Clock类提供了访问当前日期和时间的方法，Clock是时区敏感的，可以用来取代 System.currentTimeMillis() 来获取当前的微秒数。某一个特定的时间点也可以使用Instant类来表示，Instant类也可以用来创建老的java.util.Date对象。

代码如下:

```java
Clock clock = Clock.systemDefaultZone(); 
long millis = clock.millis();

Instant instant = clock.instant(); Date legacyDate = Date.from(instant); // legacy java.util.Date
```

#### **Timezones** **时区**

在新API中时区使用ZoneId来表示。时区可以很方便的使用静态方法of来获取到。 时区定义了到UTS时间的时间差，在Instant时间点对象到本地日期对象之间转换的时候是极其重要的。

代码如下:

```java
System.out.println(ZoneId.getAvailableZoneIds()); 
// prints all available timezone ids
ZoneId zone1 = ZoneId.of("Europe/Berlin"); 
ZoneId zone2 = ZoneId.of("Brazil/East"); System.out.println(zone1.getRules()); System.out.println(zone2.getRules());
// ZoneRules[currentStandardOffset=+01:00] 
// ZoneRules[currentStandardOffset=-03:00]
```

#### **LocalTime** **本地时间**

LocalTime 定义了一个没有时区信息的时间，例如 晚上10点，或者 17:30:15。下面的例子使用前面代码创建的时区创建了两个本地时间。之后比较时间并以小时和分钟为单位**计算两个时间的时间差**：

代码如下:

```java
//接着上面的zone1和zone2
LocalTime now1 = LocalTime.now(zone1); 
LocalTime now2 = LocalTime.now(zone2);
System.out.println(now1.isBefore(now2)); // false

long hoursBetween = ChronoUnit.HOURS.between(now1, now2); 
long minutesBetween = ChronoUnit.MINUTES.between(now1, now2);

System.out.println(hoursBetween); // -3 
System.out.println(minutesBetween); // -239
```

LocalTime 提供了多种工厂方法来简化对象的创建，包括解析时间字符串。

代码如下:

```java
LocalTime late = LocalTime.of(23, 59, 59); System.out.println(late); // 23:59:59

DateTimeFormatter germanFormatter = DateTimeFormatter .ofLocalizedTime(FormatStyle.SHORT) .withLocale(Locale.GERMAN);

LocalTime leetTime = LocalTime.parse("13:37", germanFormatter); 
System.out.println(leetTime); // 13:37
```

#### **LocalDate** **本地日期**

LocalDate 表示了一个确切的日期，比如 2014-03-11。该对象值是不可变的，用起来和LocalTime基本一致。下面的例子展示了如何给Date对象加减天/月/年。另外要注意的是这些对象是不可变的，操作返回的总是一个新实例。

代码如下:

```java
LocalDate today = LocalDate.now(); 
LocalDate tomorrow = today.plus(1, ChronoUnit.DAYS); 
LocalDate yesterday = tomorrow.minusDays(2);

LocalDate independenceDay = LocalDate.of(2014, Month.JULY, 4); 
DayOfWeek dayOfWeek = independenceDay.getDayOfWeek();

System.out.println(dayOfWeek); 
// FRIDAY 从字符串解析一个LocalDate类型和解析LocalTime一样简单：
```

代码如下:

```java
DateTimeFormatter germanFormatter = DateTimeFormatter .ofLocalizedDate(FormatStyle.MEDIUM) .withLocale(Locale.GERMAN);

LocalDate xmas = LocalDate.parse("24.12.2014", germanFormatter); System.out.println(xmas); // 2014-12-24
```

#### **LocalDateTime** **本地日期时间**

1、LocalDateTime 同时表示了时间和日期，相当于前两节内容合并到一个对象上了。LocalDateTime和LocalTime还有LocalDate一样，都是不可变的。

2、本地日期和时间通过now()获取到的总是以当前默认时区返回的，和旧API不同，   LocalDateTime、LocalDate和LocalTime默认严格按照**ISO 8601**规定的日期和时间格式进行打印。

> 该类完美的处理了时间问题，即使你的APP涉及国外用户，只要你们开发团队在时间格式上都遵守ISO8601格式，那么你就再也不用担心时间出错了。
>
> ISO 8601规定的日期和时间分隔符是T。**标准格式**如下：
>
> - 日期：yyyy-MM-dd
> - 时间：HH:mm:ss
> - 带毫秒的时间：HH:mm:ss.SSS
> - 日期和时间：yyyy-MM-dd'T'HH:mm:ss
> - 带毫秒的日期和时间：yyyy-MM-dd'T'HH:mm:ss.SSS

3、LocalDateTime提供了一些能访问具体字段的方法。

代码如下:

```java
LocalDateTime sylvester = LocalDateTime.of(2014, Month.DECEMBER, 31, 23, 59, 59);
//指定的日期和时间创建
DayOfWeek dayOfWeek = sylvester.getDayOfWeek(); System.out.println(dayOfWeek); 
// WEDNESDAY
Month month = sylvester.getMonth(); System.out.println(month); 
// DECEMBER
long minuteOfDay = sylvester.getLong(ChronoField.MINUTE_OF_DAY); System.out.println(minuteOfDay); 
// 1439

// eg2:
// 指定日期和时间:
LocalDate d2 = LocalDate.of(2019, 11, 30); // 2019-11-30, 注意11=11月
LocalTime t2 = LocalTime.of(15, 16, 17); // 15:16:17
LocalDateTime dt2 = LocalDateTime.of(2019, 11, 30, 15, 16, 17);
LocalDateTime dt3 = LocalDateTime.of(d2, t2);
     因为严格按照ISO 8601的格式，因此，将字符串转换为LocalDateTime就可以传入标准格式：
		LocalDateTime dt = LocalDateTime.parse("2019-11-19T15:16:17");
LocalDate d = LocalDate.parse("2019-11-19");
LocalTime t = LocalTime.parse("15:16:17");

```

4、只要附加上时区信息，就可以将其转换为一个时间点Instant对象，Instant时间点对象可以很容易的转换为老式的java.util.Date。

代码如下:

```java
Instant instant = sylvester .atZone(ZoneId.systemDefault()) .toInstant();

Date legacyDate = Date.from(instant); System.out.println(legacyDate); // Wed Dec 31 23:59:59 CET 2014
```

5、格式化LocalDateTime和格式化时间和日期一样的，除了使用预定义好的格式外，我们也可以自己定义格式：

代码如下:

```java
DateTimeFormatter formatter = DateTimeFormatter .ofPattern("MMM dd, yyyy - HH:mm");

LocalDateTime parsed = LocalDateTime.parse("Nov 03, 2014 - 07:13", formatter); 

String string = formatter.format(parsed); System.out.println(string); // Nov 03, 2014 - 07:13
```

`**和java.text.NumberFormat不一样的是新版的DateTimeFormatter是不可变的，所以它是线程安全的。**`



6、**LocalDateTime**提供了对日期和时间进行加减的非常简单的链式调用：

![image-20220606232954597](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206062329040.png)

7、**对日期和时间进行调整则使用**withXxx()**方法**，例如：withHour(15)会把10:11:12变为15:11:12：

调整年：withYear()

调整月：withMonth()

调整日：withDayOfMonth()

调整时：withHour()

调整分：withMinute()

调整秒：withSecond()

8、要判断两个LocalDateTime的先后，可以使用isBefore()、isAfter()方法，对于LocalDate和LocalTime类似：

```java
System.out.println(LocalDate.now().isBefore(LocalDate.of(2019, 11, 19)));

System.out.println(LocalTime.now().isAfter(LocalTime.parse("08:15:00")));
```

#### **ZonedDateTime**

1、表示一个带时区的日期和时间。简单地把ZonedDateTime理解成LocalDateTime加ZoneId。ZoneId是java.time引入的新的时区类

2、要创建一个ZonedDateTime对象，有以下几种方法，一种是通过now()方法返回当前时间

3、加时区两种方法：

```java
ZonedDateTime zbj = ZonedDateTime.now(); 
// 默认时区
ZonedDateTime zny = ZonedDateTime.now(ZoneId.of("America/New_York")); // 用指定时区获取当前时间


//为LocalDateTime添加一个ZoneId就变成ZonedDateTime对象
 LocalDateTime ldt = LocalDateTime.of(2019, 9, 15, 15, 16, 17);
 ZonedDateTime zbj = ldt.atZone(**ZoneId**.systemDefault());
//默认时区
 ZonedDateTime zny = ldt.atZone(**ZoneId**.of("America/New_York"));
```

4、ZonedDateTime仍然提供了plusDays()等加减操作。

![image-20220606233904248](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206062339307.png)



**Instant**：

1、计算机存储的当前时间，本质上只是一个不断递增的整数。Java提供的System.currentTimeMillis()返回的就是以毫秒表示的当前时间戳。

2、这个当前时间戳在java.time中以Instant类型表示，我们用Instant.now()获取当前时间戳

3、[Java8新特性之Instant详解](https://blog.csdn.net/corleone_4ever/article/details/104403042)

LocalDateTime，ZoneId，Instant，ZonedDateTime和long都可以互相转换：

![image-20220606231904807](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206062340308.png)

```java
/*
	转换的时候，只需要留意long类型以毫秒还是秒为单位即可。
	Instant表示高精度时间戳，
	它可以和ZonedDateTime以及long互相转换。
*/
Instant ins=Instant.ofEpochSecond(0);
ZonedDateTime zdt=ins.atZone(ZoneId.systemDefault());
DateTimeFormatter dateTimeFormatter=dtf.ofPattern("yyyy-MM-ddmm:hh:ss");
String format=dateTimeFormatter.format(zdt);
System.out.println(format);
//1970-01-0100:08:00
System.out.println(zdt);
//2019-09-16T01:32:40+08:00[Asia/Shanghai]
```



#### **简单总结：**

本地日期和时间：LocalDateTime，LocalDate，LocalTime；
带时区的日期和时间：ZonedDateTime；
时刻：Instant；
时区：ZoneId，ZoneOffset；
时间间隔：Duration。


新的用于取代SimpleDateFormat的格式化类型DateTimeFormatter

1、新API严格区分了时刻、本地日期、本地时间和带时区的日期时间，并且，对日期和时间进行运算更加方便。

2、新API修正了旧API不合理的常量设计：

- Month的范围用1~12表示1月到12月；Week的范围用1~7表示周一到周日。

例如：

`2019-10-31减去1个月得到的结果是2019-09-30 `



### 十、Annotation 注解

[Java 注解（Annotation）](https://www.runoob.com/w3cnote/java-annotation.html) 

1、Java 标注，是 JDK5.0 引入的一种注释机制。

Java 语言中的类、方法、变量、参数和包等都可以被标注。和 Javadoc 不同，Java 标注可以通过反射获取标注内容。在编译器生成类文件时，标注可以被嵌入到字节码中。Java 虚拟机可以保留标注内容，在运行时可以获取到标注内容 。 当然它也支持自定义 Java 标注。

Java 定义了一套注解，共有 7 个，3 个在 **java.lang** 中，剩下 4 个在 **java.lang.annotation** 中。

 

**作用在代码的注解是**

- @Override -     检查该方法是否是重写方法。如果发现其父类，或者是引用的接口中并没有该方法时，会报编译错误。
- @Deprecated -     标记过时方法。如果使用该方法，会报编译警告。
- @SuppressWarnings -     指示编译器去忽略注解中声明的警告。

**作用在其他注解的注解(或者说 元注解)是:**

- @Retention -     标识这个注解怎么保存，是只在代码中，还是编入class文件中，或者是在运行时可以通过反射访问。
- @Documented -     标记这些注解是否包含在用户文档中。
- @Target -     标记这个注解应该是哪种 Java 成员。
- @Inherited -     标记这个注解是继承于哪个注解类(默认 注解并没有继承于任何子类)

**从 Java 7 开始，额外添加了 3 个注解:**

- @SafeVarargs - Java     7 开始支持，忽略任何使用参数为泛型变量的方法或构造函数调用产生的警告。
- @FunctionalInterface     - Java 8 开始支持，标识一个匿名函数或函数式接口。
- @Repeatable - Java     8 开始支持，标识某注解可以在同一个声明上使用多次。

 

2、Annotion框架

![image-20220607002752583](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206070027655.png)

**组成部分：**

**(01) Annotation 就是个接口。**

"每 1 个 Annotation" 都与 "1 个 RetentionPolicy" 关联，并且与 "1～n 个 ElementType" 关联。可以通俗的理解为：每 1 个 Annotation 对象，都会有唯一的 RetentionPolicy 属性；至于 ElementType 属性，则有 1~n 个。

**(02) ElementType 是 Enum 枚举类型，它用来指定 Annotation 的类型。**

"每 1 个 Annotation" 都与 "1～n 个 ElementType" 关联。当 Annotation 与某个 ElementType 关联时，就意味着：Annotation有了某种用途。例如，若一个 Annotation 对象是 METHOD 类型，则该 Annotation 只能用来修饰方法。

eg：

**package** java.lang.annotation;

**public** **enum** ElementType {

  TYPE,        */\* 类、接口（包括注释类型）或枚举声明  \*/*

  FIELD,        */\* 字段声明（包括枚举常量）  \*/*

  METHOD,       */\* 方法声明  \*/*

  PARAMETER,      */\* 参数声明  \*/*

  CONSTRUCTOR,     */\* 构造方法声明  \*/*

  LOCAL_VARIABLE,   */\* 局部变量声明  \*/*

  ANNOTATION_TYPE,   */\* 注释类型声明 \*/*

  **PACKAGE**       */\* 包声明  \*/*

}

 

**(03) RetentionPolicy 是 Enum 枚举类型，它用来指定 Annotation 的策略。通俗点说，就是不同 RetentionPolicy 类型的 Annotation 的作用域不同。**

 

"每 1 个 Annotation" 都与 "1 个 RetentionPolicy" 关联。

- a) 若 Annotation     的类型为 SOURCE，则意味着：Annotation 仅存在于编译器处理期间，编译器处理完之后，该 Annotation 就没用了。     例如，" @Override" 标志就是一个     Annotation。当它修饰一个方法的时候，就意味着该方法覆盖父类的方法；并且在编译期间会进行语法检查！编译器处理完后，"@Override"     就没有任何作用了。
- b) 若 Annotation     的类型为 CLASS，则意味着：编译器将 Annotation 存储于类对应的 .class 文件中，它是 Annotation 的默认行为。
- c) 若 Annotation     的类型为 RUNTIME，则意味着：编译器将 Annotation 存储于 class 文件中，并且可由JVM读入。

这时，只需要记住"每 1 个 Annotation" 都与 "1 个 RetentionPolicy" 关联，并且与 "1～n 个 ElementType" 关联。

 

eg：

```java
SOURCE,       /* Annotation**信息仅存在于编译器处理期间，编译器处理完之后就没有该**Annotation**信息了*  **/
CLASS,       /** 编译器将**Annotation**存储于类对应的**.class**文件中。默认行为*  **/
RUNTIME       /* *编译器将**Annotation**存储于**class**文件中，并且可由**JVM**读入* **/


@Documented
 @Target(ElementType.TYPE)
 @Retention(RetentionPolicy.RUNTIME)
 public @interface MyAnnotation1 {
 }// 
```

 

理解：

**(01) @interface**

使用 @interface 定义注解时，意味着它实现了 java.lang.annotation.Annotation 接口，即该注解就是一个Annotation。

定义 Annotation 时，@interface 是必须的。

注意：它和我们通常的 implemented 实现接口的方法不同。Annotation 接口的实现细节都由编译器完成。通过 @interface 定义注解后，该注解不能继承其他的注解或接口。

**(02) @Documented**

类和方法的 Annotation 在缺省情况下是不出现在 javadoc 中的。如果使用 @Documented 修饰该 Annotation，则表示它可以出现在 javadoc 中。

定义 Annotation 时，@Documented 可有可无；若没有定义，则 Annotation 不会出现在 javadoc 中。

**(03) @Target(ElementType.TYPE)**

前面我们说过，ElementType 是 Annotation 的类型属性。而 @Target 的作用，就是来指定 Annotation 的类型属性。

@Target(ElementType.TYPE) 的意思就是指定该 Annotation 的类型是 ElementType.TYPE。这就意味着，MyAnnotation1 是来修饰"类、接口（包括注释类型）或枚举声明"的注解。

定义 Annotation 时，@Target 可有可无。若有 @Target，则该 Annotation 只能用于它所指定的地方；若没有 @Target，则该 Annotation 可以用于任何地方。

**(04) @Retention(RetentionPolicy.RUNTIME)**

前面我们说过，RetentionPolicy 是 Annotation 的策略属性，而 @Retention 的作用，就是指定 Annotation 的策略属性。

@Retention(RetentionPolicy.RUNTIME) 的意思就是指定该 Annotation 的策略是 RetentionPolicy.RUNTIME。这就意味着，编译器会将该 Annotation 信息保留在 .class 文件中，并且能被虚拟机读取。

定义 Annotation 时，@Retention 可有可无。若没有 @Retention，则默认是 RetentionPolicy.CLASS。

3、Annotation作用

Annotation 是一个辅助类，它在 Junit、Struts、Spring 等工具框架中被广泛使用。

**1）编译检查**

Annotation 具有"让编译器进行编译检查的作用"。

例如，@SuppressWarnings, @Deprecated 和 @Override 都具有编译检查作用。

(01) 关于 @SuppressWarnings 和 @Deprecated，已经在"第3部分"中详细介绍过了。这里就不再举例说明了。

(02) 若某个方法被 @Override 的标注，则意味着该方法会覆盖父类中的同名方法。如果有方法被 @Override 标示，但父类中却没有"被 @Override 标注"的同名方法，则编译器会报错。

**2) 在反射中使用 Annotation**

在反射的 Class, Method, Field 等函数中，有许多于 Annotation 相关的接口。

这也意味着，我们可以在反射中解析并使用 Annotation。

**3) 根据 Annotation 生成帮助文档**

通过给 Annotation 注解加上 @Documented 标签，能使该 Annotation 标签出现在 javadoc 中。

**4) 能够帮忙查看查看代码**

通过 @Override, @Deprecated 等，我们能很方便的了解程序的大致结构。

另外，我们也可以通过自定义 Annotation 来实现一些功能。

 

注：框架 = 注解 + 反射 + 设计模式



### 十一、Stream流和方法引用

[Java8 parallelStream浅析](https://zhuanlan.zhihu.com/p/43039062)    [使用Java 8新增的Stream操作Collection集合 ](http://c.biancheng.net/view/6805.html)  [JDK8 Stream ](https://blog.csdn.net/Al_assad/article/details/82356845)  

- **stream()** − 为集合创建串行流。

- **parallelStream()** − 为集合创建并行流。

- 1. **sream顺序输出，而parallelStream 无序输出；parallelStream      执行耗时是 stream 的五分之一。**
  2. **parallelStream 获得的相对较好的执行性能，但是如果cpu紧张，不会带来性能的优化，因为要频发的线程切换。**
  3. **stream.parallel.forEach()中执行的操作并非线程安全。**
  4. **如果需要线程安全，可以把集合转换为同步集合，即：Collections.synchronizedList(new ArrayList<>())。**

1、Stream便容易想到I/O Stream，在Java 8中，得益于Lambda所带来的函数式编程，引入了一个全新的Stream概念，用于解决已有集合类库既有的弊端。



2、传统的集合遍历是顺序遍历，一个**迭代器对象只能便利一次**。所有用Stream流优化。

步骤：

当使用一个流的时候，通常包括三个基本步骤：**获取一个数据源（source）→ 数据转换→执行操作获取想要的结果，每次转换原有 Stream 对象不改变，返回一个新的 Stream 对象（可以有多次转换）**，这就允许对其操作可以像链条一样排列，变成一个管道。 



3、流式思想：类似于生产线，对获取到的流进行，过滤、映射、计数、跳步…..；

filter 、 map 、 skip 都是在对函数模型进行操作，集合元素并没有真正被处理。只有当终结方法 count 执行的时候，整个模型才会按照指定策略执行操作。而这得益于Lambda的延迟执行特性。 

备注：“Stream流”其实是一个集合元素的函数模型，它并不是集合，也不是数据结构，其本身并不存储任何 元素（或其地址值）。 



4、获取数据流：java.util.stream.Stream 是Java 8新加入的

#### 最常用的流接口

a、**所有的 Collection 集合都可以通过 stream 默认方法获取流：** 

**Collection**接口中有default方法Stream（）获取流（所有子类都有该方法）。

Stream stream = list.stream()

b、Stream 接口的**静态方法 of** 可以获取数组对应的流。 

Stream stream = Stream.of(array);

c、**Map接口 **K-V的数据结构，获取流要分开（key、value、entry）。

Stream keyStream = map.keySet().stream(); 

Stream valueStream = map.values().stream(); 

Stream<Map.Entry> entryStream = map.entrySet().stream();

 

#### **流方法**：

Stream 提供了大量的方法进行聚集操作。

- 中间方法：中间操作允许流保持打开状态，并允许直接调用后续方法。上面程序中的     map() 方法就是中间方法。中间方法的返回值是另外一个流。
- 末端方法：末端方法是对流的最终操作。当对某个     Stream 执行末端方法后，该流将会被“消耗”且不再可用。上面程序中的 sum()、count()、average() 等方法都是末端方法。

 

流的方法还有如下两个特征。

- 有状态的方法：这种方法会给流增加一些新的属性，比如元素的唯一性、元素的最大数量、保证元素以排序的方式被处理等。有状态的方法往往需要更大的性能开销。
- 短路方法：短路方法可以尽早结束对流的操作，不必检查所有的元素。

 

####  **Stream 常用的中间方法。**

|                           **方法**                           | **说明**                                                     |
| :----------------------------------------------------------: | ------------------------------------------------------------ |
|               **filter**(Predicate predicate)                | 过滤 Stream 中所有不符合 predicate 的元素                    |
| **mapToXxx**(ToXxxFunction mapper)  map（Function mapper）：映射 | **使用 ToXxxFunction  对流中的元素执行一对一的转换，该方法返回的新流中包含了 ToXxxFunction 转换生成的所有元素。**  **当前流****T****类型转化为****R****类型** |
|                    peek(Consumer  action)                    | 依次对每个元素执行一些操作，该方法返回的流与原有流包含相同的元素。该方法主要用于调试。 |
|                          distinct()                          | 该方法用于排序流中所有重复的元素（判断元素重复的标准是使用 equals() 比较返回 true）。这是一个有状态的方法。 |
|                           sorted()                           | 该方法用于保证流中的元素在后续的访问中处于有序状态。这是一个有状态的方法。 |
|                   **limit**(long maxSize)                    | 该方法用于保证对该流的后续访问中最大允许访问的元素个数。这是一个有状态的、短路方法。 |
|                    **skip**（**long n**）                    | 希望跳过前几个元素                                           |
|   **Static** **concat**（**Stream a** **，**Stream b**）**   | 两个流，希望合并成为一个流                                   |

 

####  **Stream 常用的末端方法。**

|               **方法**               | **说明**                                                   |
| :----------------------------------: | ---------------------------------------------------------- |
| **forEach**(**Consumer** **action)** | 遍历流中所有元素，对每个元素执行action                     |
|            **toArray()**             | 将流中所有元素转换为一个数组                               |
|               reduce()               | 该方法有三个重载的版本，都用于通过某种操作来合并流中的元素 |
|                min()                 | 返回流中所有元素的最小值                                   |
|                max()                 | 返回流中所有元素的最大值                                   |
|             **count()**              | 返回流中所有元素的数量                                     |
|    anyMatch(Predicate  predicate)    | 判断流中是否至少包含一个元素符合 Predicate 条件。          |
|    allMatch(Predicate predicate)     | 判断流中是否每个元素都符合 Predicate 条件                  |
|    noneMatch(Predicate predicate)    | 判断流中是否所有元素都不符合 Predicate 条件                |
|             findFirst()              | 返回流中的第一个元素                                       |
|              findAny()               | 返回流中的任意一个元素                                     |

 

```java
Random random = new Random(); 
//只要是个排序，遍历输出
random.ints().limit(10).sorted().forEach(System.out::println);

List<String> strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
 // 获取空字符串的数量 
long count = strings.parallelStream().filter(string -> string.isEmpty()).count();

Stream streamOne = one.stream().filter(s ‐> s.length() == 3).limit(3);

Stream streamTwo = two.stream().filter(s ‐> s.startsWith("张")).skip(2); 

Stream.concat(streamOne, streamTwo).map(Person::new).forEach(System.out::println);
```



#### **方法引用**

1、在lambda表达式基础上简化，避免冗余，其中的双冒号 :: 写法，这被称为“方法引用”，而双冒号是一种新的语法（代替lambda写法）。

2、主要处理，操作没有在lambda进行操作，就输出，例如：：拿到参数之后经Lambda之手，继而传递给 System.out.println 方法去处理

Lambda要表达的函数方案已经存在于某个方 法的实现中，那么则可以通过双冒号来引用该方法作为Lambda的替代者。

用法：

Lambda表达式写法： s -> System.out.println(s); 

方法引用写法： System.out::println

 

Lambda表达式： n -> Math.abs(n) 

方法引用： Math::abs

 

Lambda表达式： () -> super.sayHello() 

方法引用： super::sayHello

 

lambda表达式： () -> this.buyHouse() 

方法引用： this::buyHouse 

 

Lambda表达式： name -> new Person(name) 

方法引用： Person::new

 

Lambda表达式： length -> new int[length] 

方法引用： int[]::new
