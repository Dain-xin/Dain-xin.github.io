---
title: UDP协议
date: 2022-06-18 23:10:37
categories:
- JAVA
- 高级
- 网络编程
---

[Java UDP通信：Java DatagramSocket类和DatagramPacket类](http://c.biancheng.net/view/1203.html)

 

# **UDP**协议：

在 TCP/IP 协议的传输层除了一个 TCP 协议之外，还有一个 UDP 协议。UDP 协议是用户数据报协议的简称，也用于网络数据的传输。虽然 UDP 协议是一种不太可靠的协议，但有时在需要较快地接收数据并且可以忍受较小错误的情况下，UDP 就会表现出更大的优势。

 

**UDP 协议发送数据的步骤。**

1. 使用     DatagramSocket() 创建一个数据包套接字。
2. 使用 DatagramPacket()     创建要发送的数据包。
3. 使用 DatagramSocket     类的 send() 方法发送数据包。

 

**接收 UDP 数据包的步骤如下：**

1. 使用     DatagramSocket 创建数据包套接字，并将其绑定到指定的端口。
2. 使用 DatagramPacket     创建字节数组来接收数据包。
3. 使用 DatagramPacket     类的 receive() 方法接收 UDP 包。

 

## **DatagramPacket** 类

java.**net** 包中的 **DatagramPacket** 类用来表示数据报包，数据报包用来实现无连接包投递服务。

每条报文仅根据该包中包含的信息从一台机器路由到另一台机器。

**从一台机器发送到另一台机器的多个包可能选择不同的路由，也可能按不同的顺序到达。**

 

 

### **DatagramPacket的构造方法：**

| **构造方法**                                                 | **说明**                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| DatagramPacket(byte[] buf,int length)                        | **构造  DatagramPacket，用来接收长度为 length 的数据包。**   |
| DatagramPacket(byte[] buf,int offset,  int length)           | 构造 DatagramPacket，用来接收长度为 length 的包，在缓  冲区中指定了偏移量。 |
| **DatagramPacket(byte[] buf,int  length,**  **InetAddress address,int port)** | **构造  DatagramPacket，用来将长度为 length 的包发送到指**  **定主机上的指定端口。** |
| DatagramPacket(byte[] buf,int length,  SocketAddress address) | 构造数据报包，用来将长度为 length 的包发送到指定主机上  的指定端口。 |
| DatagramPacket(byte[] buf,int offset,  int length,InetAddress address,int port) | 构造 DatagramPacket，用来将长度为 length 偏移量为 offset  的包发送到指定主机上的指定端口。 |
| DatagramPacket(byte[] buf,int offset,  int length,SocketAddress address) | 构造数据报包，用来将长度为 length、偏移量为 offset 的包发  送到指定主机上的指定端口。 |

 

 

 

 

###  **DatagramPacket的常用方法：**

 

| **方法**                                               | **说明**                                                     |
| ------------------------------------------------------ | ------------------------------------------------------------ |
| **InetAddress getAddress()**                           | **返回某台机器的 IP 地址，此数据报将要发往该机器或者**  **从该机器接收。** |
| **byte[] getData()**                                   | **返回数据缓冲区。**                                         |
| int getLength()                                        | 返回将要发送或者接收的数据的长度。                           |
| int getOffset()                                        | 返回将要发送或者接收的数据的偏移量。                         |
| int getPort()                                          | 返回某台远程主机的端口号，此数据报将要发往该主机或  者从该主机接收。 |
| **getSocketAddress()**                                 | **获取要将此包发送或者发出此数据报的远程主机的**  **SocketAddress（通常为  IP地址+端口号）。** |
| void setAddress(InetAddress addr)                      | 设置要将此数据报发往的目的机器的IP地址。                     |
| void setData(byte[] buf)                               | 为此包设置数据缓冲区。                                       |
| void setData(byte[] buf,int offset,  int length)       | 为此包设置数据缓冲区。                                       |
| void setLength(int length)                             | 为此包设置长度。                                             |
| void setPort(int port)                                 | 设置要将此数据报发往的远程主机的端口号。                     |
| **void  setSocketAddress(SocketAddress**  **address)** | **设置要将此数据报发往的远程主机的**  **SocketAddress（通常为 IP地址+端口号）。** |

## **DatagramSocket 类**

**DatagramSocket** 类用于表示发送和接收数据报包的套接字。

**数据报包套接字是包投递服务的发送或接收点。**

每个在数据报包套接字上发送或接收的包都是单独编址和路由的。

从一台机器发送到另一台机器的多个包可能选择不同的路由，也可能按不同的顺序到达。

 

### **DatagramSocket 类的常用构造方法：**

| **构造方法**                             | **说明**                                                     |
| ---------------------------------------- | ------------------------------------------------------------ |
| DatagramSocket()                         | 构造数据报包套接字并将其绑定到本地主机上任何可用的端口。     |
| **DatagramSocket(int port)**             | 创建数据报**包套接字并将其绑定到本地主机上的****指定端口****。** |
| DatagramSocket(int portJnetAddress addr) | 创建数据报包套接字，将其绑定到指定的本地地址。               |
| DatagramSocket(SocketAddress bindaddr)   | 创建数据报包套接字，将其绑定到指定的本地套接字地址。         |

 

### **DatagramSocket 类的常用方法：**

| **方法**                                       | **说明**                                          |
| ---------------------------------------------- | ------------------------------------------------- |
| void **bind**(SocketAddress addr)              | 将此 DatagramSocket 绑定到特定的地址和端口。      |
| void close()                                   | 关闭此数据报包套接字。                            |
| void **connect**(InetAddress address,int port) | 将套接字连接到此套接字的远程地址。                |
| void connect(SocketAddress addr)               | 将此套接子连接到远程套接子地址（IP地址+端口号）。 |
| void **disconnect**()                          | 断开套接字的连接。                                |
| InetAddress getInetAddress()                   | 返回此套接字连接的地址。                          |
| InetAddress **getLocalAddress**()              | 获取套接字绑定的本地地址。                        |
| **int getLocalPort()**                         | 返回此套接字绑定的本地主机上的端口号。            |
| int getPort()                                  | 返回此套接字的端口。                              |

 

 

 

 

# **例题：**

## **SendDemo类：**

1、创建套用字对象（DatagramSocket），然后while：可持续发送信息，Scanner类输入信息；

2、InetAddress address=InetAddress.*getByName*("127.0.0.1")：创建地址。

3、创建数据包对象（DatagramPacket类），构造方法（发送数据：byte[ ]，发送长度，发送地址：InetAddress，发送端口：port）。

4、用套用子对象的send（数据包（DatagramPacket））方法。

5、Thread.sleep(1000);可以休眠。

 

## **ReceiveDemo类**

1、创建套用字对象，构造方法（端口号）。while（true）：可持续接受数据。

2、创建数据包对象，构造方法（发送的数据，数据的长度）。

3、套用子对象的recive（数据包）方法。收到的数据都在数据包里面。

4、输出即可，**String** **str=new** **String(datagramPacket.getData(),**  **0,**  **datagramPacket.getLength()**  **);**