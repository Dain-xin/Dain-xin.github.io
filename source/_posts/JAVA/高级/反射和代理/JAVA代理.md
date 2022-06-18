---
title: JAVA代理
date: 2022-06-03 22:52:37
categories:
- JAVA
- 高级
- 反射和代理
---

#  Java代理的几种实现方式 



第一种:静态代理,只能静态的代理某些类或者某些方法,不推荐使用,功能比较弱,但是编码简单

第二种:动态代理,包含Proxy代理和CGLIB动态代理

## Proxy代理是JDK内置的动态代理

​         特点:面向接口的,不需要导入三方依赖的动态代理,可以对多个不同的接口进行增强,通过反射读取注解时,只能读取到接口上的注解

​         原理:面向接口,只能对实现类在实现接口中定义的方法进行增强

定义接口和实现

```java
package com.proxy;

public interface UserService {
    public String getName(int id);

    public Integer getAge(int id);
}
```

```java
package com.proxy;

public class UserServiceImpl implements UserService {
    @Override
    public String getName(int id) {
        System.out.println("------getName------");
        return "riemann";
    }

    @Override
    public Integer getAge(int id) {
        System.out.println("------getAge------");
        return 26;
    }
}
```

```java
package com.proxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

public class MyInvocationHandler implements InvocationHandler {

    public Object target;

    MyInvocationHandler() {
        super();
    }

    MyInvocationHandler(Object target) {
        super();
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        if ("getName".equals(method.getName())) {
            System.out.println("++++++before " + method.getName() + "++++++");
            Object result = method.invoke(target, args);
            System.out.println("++++++after " + method.getName() + "++++++");
            return result;
        } else {
            Object result = method.invoke(target, args);
            return result;
        }
    }
}
```

```java
package com.proxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Proxy;

public class Main1 {
    public static void main(String[] args) {
        UserService userService = new UserServiceImpl();
        InvocationHandler invocationHandler = new MyInvocationHandler(userService);
        UserService userServiceProxy = (UserService) Proxy.newProxyInstance(userService.getClass().getClassLoader(),
                userService.getClass().getInterfaces(),invocationHandler);
        System.out.println(userServiceProxy.getName(1));
        System.out.println(userServiceProxy.getAge(1));
    }
}
```



## CGLIB动态代理

​        特点:面向父类的动态代理,需要导入第三方依赖

​        原理:面向父类,底层通过子类继承父类并重写方法的形式实现增强

Proxy和CGLIB是非常重要的代理模式,是springAOP底层实现的主要两种方式

CGLIB的核心类：
net.sf.cglib.proxy.Enhancer – 主要的增强类
net.sf.cglib.proxy.MethodInterceptor – 主要的方法拦截类，它是Callback接口的子接口，需要用户实现
net.sf.cglib.proxy.MethodProxy – JDK的java.lang.reflect.Method类的代理类，可以方便的实现对源对象方法的调用,如使用：
Object o = methodProxy.invokeSuper(proxy, args);//虽然第一个参数是被代理对象，也不会出现死循环的问题。

net.sf.cglib.proxy.MethodInterceptor接口是最通用的回调（callback）类型，它经常被基于代理的AOP用来实现拦截（intercept）方法的调用。这个接口只定义了一个方法
public Object intercept(Object object, java.lang.reflect.Method method,
Object[] args, MethodProxy proxy) throws Throwable;

第一个参数是代理对像，第二和第三个参数分别是拦截的方法和方法的参数。原来的方法可能通过使用java.lang.reflect.Method对象的一般反射调用，或者使用 net.sf.cglib.proxy.MethodProxy对象调用。net.sf.cglib.proxy.MethodProxy通常被首选使用，因为它更快。

```java
package com.proxy.cglib;

import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;
import java.lang.reflect.Method;
 
public class CglibProxy implements MethodInterceptor {
    @Override
    public Object intercept(Object o, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        System.out.println("++++++before " + methodProxy.getSuperName() + "++++++");
        System.out.println(method.getName());
        Object o1 = methodProxy.invokeSuper(o, args);
        System.out.println("++++++before " + methodProxy.getSuperName() + "++++++");
        return o1;
    }
}

```

```java
package com.proxy.cglib;
 
import com.test3.service.UserService;
import com.test3.service.impl.UserServiceImpl;
import net.sf.cglib.proxy.Enhancer;
 
public class Main2 {
    public static void main(String[] args) {
        CglibProxy cglibProxy = new CglibProxy();
 
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(UserServiceImpl.class);
        enhancer.setCallback(cglibProxy);
 
        UserService o = (UserService)enhancer.create();
        o.getName(1);
        o.getAge(1);
    }
}
```

# 动态代理

运行期动态创建一个interface实例的方法如下：
	1. 定义一个InvocationHandler实例，它负责实现接口的方法调用；
	2. 通过Proxy.newProxyInstance()创建interface实例，它需要3个参数：
	3. 使用的ClassLoader，通常就是接口类的ClassLoader；
	4. 需要实现的接口数组，至少需要传入一个接口进去；
	5. 用来处理接口方法调用的InvocationHandler实例。
	6. 将返回的Object强制转型为接口。
	

注：
动态代理是通过Proxy创建代理对象，然后将接口方法“代理”给InvocationHandler完成的。
invoke（Object obj, Object …    args）原方法声明为private,则需要在调用此invoke()方法前，

显式调用 方法对象的setAccessible(true)方法，将可访问private的方法。
Method和Field、Constructor对象都有setAccessible()方法。



**动态代理步骤：**

1. **创建一个实现接口**InvocationHandler**的类，它必须实现invoke方 法，以完成代理的具体操作。**
2. **创建被代理的类以及接口**
3. **通过Proxy的静态方法**  **newProxyInstance(ClassLoader loader, Class[] interfaces,     InvocationHandler h)** **创建 一个Subject接口代理**



## 实例代码



![image-20220618173719039](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206181737187.png)

# **静态代理：**

特征：代理类和目标 对象的类都是在编译期间确定下来，不利于程序的扩展。每一个代 理类只能为一个接口服务，这样一来程序开发中必然产生过多的代理。

## **代理模式理解：**

使用一个代理将对象包装起来, 然后用该代理对象取代原始对象。任何对原 始对象的调用都要通过代理。代理对象决定是否以及何时将方法调用转到原始对象上。

![image-20220618173520065](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206181735135.png)





# 补充：

动态代理：JDK方法

![image-20220618174022806](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206181740947.png)

CJLIB方法：动态代理。

![image-20220618174056577](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206181740704.png)
