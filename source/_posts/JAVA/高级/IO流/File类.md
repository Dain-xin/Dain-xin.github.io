---
title: File类
date: 2022-06-18 17:52:37
categories:
- JAVA
- 高级
- IO流
---

# File类：

 

**Java.io.File 类是文件和目录路径名的抽象表示，主要用于文件和目录的创建、查找和删除等操作。**

 

## **构造方法：**

| public  File(String pathname) ：             | 通过将给定的路径名字符串转换为抽象路径名来创建新的  File实例。 |
| -------------------------------------------- | ------------------------------------------------------------ |
| public  File(String parent, String child) ： | 从父路径名字符串和子路径名字符串创建新的  File实例。         |
| public  File(File parent, String child) ：   | 从父抽象路径名和子路径名字符串创建新的  File实例。           |

```java
String pathname="a:\\b\\c";
String pathname2="\\d\\e.text";
String pathname3="\\kk\\qq";
File file=newFile(pathname);
File file1=newFile(pathname,pathname2);
File file2=newFile(file,pathname3);
```

## **获取方法：**

| public String getAbsolutePath()  ：  public File getAbsoluteFile（）： | 返回此File的绝对路径名字符串。   返回此抽象路径名的绝对路径名形式（File类型）。 |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| public  String getPath() ：                                  | 将此File转换为路径名字符串。                                 |
| public  String getName() ：                                  | 返回由此File表示的文件或目录的名称。                         |
| public  long length() ：                                     | 返回由此File表示的文件的长度。                               |

 

理解：

绝对路径：从盘符开始的路径，这是一个完整的路径。

相对路径：相对于项目目录的路径。

length：目录的返回值不唯一（不确定）；但是对于文件返回文件字节数。

 

## **判断方法：**

| public  boolean exists() ：      | 此File表示的文件或目录是否实际存在。 |
| -------------------------------- | ------------------------------------ |
| public  boolean isDirectory() ： | 此File表示的是否为目录。             |
| public  boolean isFile() ：      | 此File表示的是否为文件。             |

 

## **创建与删除功能：**

| public  boolean createNewFile() ： | 当且仅当具有该名称的文件尚不存在时，创建一个新的空文件。     |
| ---------------------------------- | ------------------------------------------------------------ |
| public  boolean delete() ：        | 删除由此File表示的文件或目录。**表示目录，则目录必须为空才能删除。** |
| public  boolean mkdir() ：         | 创建由此File表示的目录。                                     |
| public  boolean mkdirs() ：        | 创建由此File表示的目录，包括任何必需但不存在的父目录。       |

 

## **遍历File方法：**

| public String[] list() ：    | 返回一个String数组，表示该File目录中的所有子文件或目录。     |
| ---------------------------- | ------------------------------------------------------------ |
| public File[] listFiles() ： | 返回**一个****File数组**，表示该File目录中的所有的子文件或目录。 |

 

### 理解：

调用listFiles方法的File对象，表示的必须是实际存在的目录，否则返回null，无法进行遍历。空文件夹长度是1.

空文件报错（**java.lang.NullPointerException**）

 

## **拓展方法：**

 

| **String** getParent()               | 返回此抽象路径名父目录的路径名字符串；如果此路径名没有指定父目录，则返回  null。 |
| ------------------------------------ | ------------------------------------------------------------ |
| **File** getParentFile()             | 返回此抽象路径名父目录的抽象路径名；如果此路径名没有指定父目录，则返回  null。 |
| boolean isAbsolute()                 | 测试此抽象路径名是否为绝对路径名。                           |
| boolean isHidden()                   | 测试此抽象路径名指定的文件是否是一个隐藏文件。               |
| long lastModified()                  | 返回此抽象路径名表示的文件最后一次被修改的时间。             |
| String[] list(FilenameFilter filter) | 返回一个字符串数组，这些字符串指定此抽象路径名表示的目录中满足指定过滤器的文件和目录。 |
| File[] listFiles(FileFilter filter)  | 返回抽象路径名数组，这些路径名表示此抽象路径名表示的目录中满足指定过滤器的文件和目录。 |

 

```java
	System.out.println(file.getAbsoluteFile().getName());
	System.out.println(file.getAbsolutePath());
	long l=file.lastModified();
	Date date=newDate(l);
System.out.println(date.toLocaleString());
```

# 递归：

指在当前方法内调用自己的本身方法。

递归的分类: 

递归分为两种，直接递归和间接递归。 直接递归称为方法自身调用自己。 间接递归可以A方法调用B方法，B方法调用C方法，C方法调用A方法。 

注意事项： 

递归一定要有条件限定，保证递归能够停止下来，否则会发生栈内存溢出。

在递归中虽然有限定条件，但是递归次数不能太多。

否则也会发生栈内存溢出。 

构造方法,禁止递归。

```java
		/*2.4
		递归打印多级目录
		分析：多级目录的打印，就是当目录的嵌套。遍历之前，
		无从知道到底有多少级目录，所以我们还是要使用递归实现
		*/
		public static int number=0;
		public static int number2=0;
		
		public static void main(String[]args){
			Filefile=newFile("d:桌面");
			printFile(file);
			System.out.println("---------------");
			System.out.println(file.getPath()+":详情");
			System.out.println("文件个数："+number);
			System.out.println("文件夹个数："+number2);
		}
		
		public static void printFile(Filefile){
			File[]files=file.listFiles();
			if(files!=null){
			for(Filef:files){
			if(f.isFile()){
				System.out.println("文件："+f.getName());
				number++;
			}else{
				System.out.println("文件夹："+f.getName());
				number2++;
				printFile(f);
				}
			}
		}

```

 **理解：个人感觉递归既然有就一定有他的简便存在的意义。**

**这里就发现了，不需要写复杂的循环检查去实现，只需要一次检查，递归调用，就可以实现一层一层调用的关系。这里就是，调用**printFile方法，在文件夹检查位置再次调用，这样在检查到文件夹，就会以这个目录传参，继续遍历，直到没有文件夹，就返回上一级遍历下一个文件夹，这样一级一级目录最后到母目录遍历完结束。

```
String：  boolean endwith(String tiaojian)  判断文件结尾是否符合条件
```

**对过滤器的理解：因为**FileFilter 接口只有一个**accept**（）所以可以直接使用**lambda**方法。

![image-20220618175804732](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206181758883.png)

```java
//lambda表达式：
File[] files = file.listFiles(
    (pathname)->{
                    return pathname.getName().endsWith(".java")||pathname.isDirectory();
                });
				
File[] files = file.listFiles(
    pathname->pathname.getName().endsWith(".java")||pathname.isDirectory()
);
```

