---
title: TCP协议
date: 2022-06-18 23:05:37
categories:
- JAVA
- 高级
- 网络编程
---

# **TCP**通讯程序

TCP通信能实现两台计算机之间的数据交互，通信的两端，要严格区分为客户端（Client）与服务端（Server）。 

\1. 服务端程序，需要事先启动，等待客户端的连接。 

\2. 客户端主动连接服务器端，连接成功才能通信。服务端不可以主动连接客户端。

 

提供了两个类用于实现TCP通信程序： 

\1. 客户端： **java.net.Socket** 类表示。创建 Socket 对象，向服务端发出连接请求，服务端响应请求，两者建立连接开始通信。 

\2. 服务端： **java.net.ServerSocket** 类表示。创建 ServerSocket 对象，相当于开启一个服务，并等待客户端的连接。

 

# **Socket**类：

该类实现客户端套接字，套接字指的是两台设备之间通讯的端点。

| **Socket(String host, int port) :** | **创建套接字对象并将其连接到指定主机上的指定端口号。** |
| ----------------------------------- | ------------------------------------------------------ |
|                                     |                                                        |

如果指定的host是null ，则相当于指定地址为回送地址。

 

| **InputStream** getInputStream() ：   | 返回此套接字的输入流。  如果此Scoket具有相关联的通道，则生成的InputStream 的所有操作也关联该通道。 关闭生成的InputStream也将关闭相关的Socket。 |
| ------------------------------------- | ------------------------------------------------------------ |
| **OutputStream** getOutputStream() ： | 返回此套接字的输出流。  如果此Scoket具有相关联的通道，则生成的OutputStream 的所有操作也关联该通道。  关闭生成的OutputStream也将关闭相关的Socket。 |
| void **close**() ：                   | 关闭此套接字。  一旦一个socket被关闭，它不可再使用。 关闭此socket也将关闭相关的InputStream和OutputStream 。 |
| void **shutdownOutput**() ：          | 禁用此套接字的输出流。  任何先前写出的数据将被发送，随后终止输出流。 |
| void **setTimeout**(int  timeOut):    | 设置连接超时的时长                                           |

 

# **ServerSocket类**

这个类实现了服务器套接字，该对象等待通过网络的请求。

| ServerSocket(int  port) ： | 使用该构造方法在创建ServerSocket对象时，就可以将其绑定到一个指定的端口号上，参数port就是**端口**号。 |
| -------------------------- | ------------------------------------------------------------ |
|                            |                                                              |

成员方法：

| **Socket**  accept() ： | 侦听并接受连接，**返回一个新的Socket对象**，用于和客户端实现通信。该方法会一直阻塞直到建立连接。 |
| ----------------------- | ------------------------------------------------------------ |
|                         |                                                              |

 

 

# **TCP**网络程序：

\1.【服务端】启动,创建ServerSocket对象，等待连接。 

\2. 【客户端】启动,创建Socket对象，请求连接。 

\3. 【服务端】接收连接,调用accept方法，并返回一个Socket对象。 

\4. 【客户端】Socket对象，获取OutputStream，向服务端写出数据。 

\5. 【服务端】Scoket对象，获取InputStream，读取客户端发送的数据。

\6. 【服务端】Socket对象，获取OutputStream，向客户端回写数据。 

\7. 【客户端】Scoket对象，获取InputStream，解析回写数据。 

\8. 【客户端】释放资源，断开连接。 

![image-20220618230509919](https://blog-images-djx.oss-cn-hangzhou.aliyuncs.com/img/202206182305024.png)

# 模拟TCP网络协议

![image-20220618230611497](../../../../../../../../../AppData/Roaming/Typora/typora-user-images/image-20220618230611497.png)