---
title: Thrift
date: 2022-09-01 12:00:00
categories: ["学习"]
tags: ["复习笔记","计算机网络","Thrift"]
mathjax: true
---

## 一、什么是Thrift？

- thrift是一种描述语言和二进制通讯协议，它被用来定义和创建跨语言的服务。它被当作一个远程过程调用框架来使用，是由Facebook为“大规模语言服务开发”而开发的。
- 它通过一个代码生成引擎联合了一个软件栈，来创建不同程度的、无缝的跨平台高效服务，可以使用C#、C++（基于POSIX兼容系统）、Cappuccino、Cocoa、Delphi、Erlang、Go、Haskell、Java、Node.js、OCaml、Perl、PHP、Python、Ruby和Smalltalk。
- 虽然之前是由Facebook开发的，但它现在是Apache软件基金会的开源项目了。

<!--more-->

## 二、架构

### 架构模型

Thrift包含一套完整的栈来创建客户端和服务端程序。顶层部分是由Thrift定义生成的代码IDL。而服务则由这个文件客户端和处理器代码生成。在生成的代码里会创建不同于内建类型的数据结构、并将其作为结果发送。协议和传输层是运行时库的一部分。有了Thrift，就可以定义一个服务或改变通讯和传输协议，而无需重新编译代码。除了客户端部分之外，Thrift还包括服务器基础设施来集成协议和传输，如阻塞、非阻塞及多线程服务器。栈中作为I/O基础的部分对于不同的语言则有不同的实现。如下图所示：

<img src="https://img-blog.csdn.net/2018081919583516?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3prcF9qYXZh/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70" alt="thrift架构" style="zoom:50%;" />

Thrift**软件栈**分层**从下向上**分别为：**传输层**（Transport Layer）、**协议层**（Protocol Layer）、**处理层**（Processor Layer）和**服务层**（Server Layer）。

- **传输层**：传输层负责直接从网络中**读取**和**写入**数据，它定义了**具体的网络传输协议**：比如TCP/IP传输等。
- **协议层**：协议层定义了**数据传输格式**，负责网络传输数据的**序列化**和**反序列化**：比如说JSON、XML、二进制数据等。
- **处理层**：处理层是由具体的IDL（**接口描述语言**）生成的，封装了具体的**底层网络传输和序列化方式**，并委托给用户实现的Handler进行处理。
- **服务层**：整合上述组件，提供具体的网络线程/IO服务模型，形成最终的服务。

### 支持的通讯协议

- TBinaryProtocol
  - 一种简单的二进制格式，简单，但没有为空间效率而优化。比文本协议处理起来更快，但更难于调试。
- TCompactProtocol
  - 更紧凑的二进制格式，处理起来同样高校
- TDebugProtocol
  - 一种人类可读的文本格式，用来协助调试。
- TDenseProtocol
  - 与TcompactProtocol类似，将传输数据的原信息剥离。
- TJSONProtocol
  - 使用JSON对数据编码
- TSimpleJSONProtocol
  - 一种只写协议，它不能被Thrift解析

### 支持的传输协议

- TFileTransport
  - 该传输协议会写文件
- TFramedTransport
  - 当使用一个非阻塞服务器时，要求使用这个传输协议。它按帧来发送数据，其中每一帧的开头是长度信息。
- TMemoryTransport
  - 使用存储器映射输入输出（Java的实现用了一个简单的ByteArrayOutputStream）
- TSocket
  - 使用阻塞的套接字I/O来传输。
- TZlibTransport
  - 用Zlib进行压缩，用于连接另一个传输协议

### 支持的服务类型

- TNonblockingServer
  - 一个多线程服务器，它使用非阻塞I/O（Java的实现使用了NIO通道）。TFramedTransport必须跟这个服务器配套使用。
- TSimpleServer
  - 一个单线程服务器，它使用标准的阻塞I/O。测试时很有用。
- TthreadPoolServer
  - 一个多线程服务器，它使用标准的阻塞I/O
- THsHaServer
  - YHsHa引入了线程池去处理（需要使用TramedTransport数据传输方式），其模型把读写任务放到线程池去处理，Half-sync/Half-async的处理模式，Half-sync是在处理IO时间上（accept/read/write io），Half-async用于handle对RPC的同步处理
- TThreadedSelectorServer
  - 多线程选择器服务器端，对THsHaServer在异步IO模型上进行增强

## 三、优势

- 跟一些替代选择，比如SOAP相比，跨语言序列化的代价更低，因为它使用二进制格式。
- 它有一个又瘦又干净的库，没有编码框架，没有XML配置文件。
- 绑定感觉很自然。例如，Java使用java.util.ArrayList;C++使用std::vectorstd::string.
- 应用层通讯格式与序列化层通讯格式是完全分离的。它们都可以独立修改。
- 预定义的序列化格式包括：二进制格式、对HTTP友好的格式，以及紧凑的二进制格式。
- 兼作跨语言文件序列化。
- 协议使用软版本号机制软件版本管理（需要解释）。Thrift不要求一个中心化和显式的版本号机制，例如主版本号/次版本号。松耦合的团队可以轻松控制RPC调用的演进。
- 没有构建依赖也不含非标准化的软件。不存在不兼容的软件许可证混用的情况

## 四、Thrift的数据类型

Thrift脚本可定义的数据类型包括以下几种类型：

**1.基本类型：**

- bool：布尔值
- byte：8位有符号整数
- i16：16位有符号整数
- i32：32位有符号整数
- i64：64位有符号整数
- double：64位浮点数
- string：UTF-8编码的字符串
- binary：二进制串

**2.结构体类型：**

- struct：定义的结构体对象

**3.容器类型：**

- list：有序元素列表
- set：无序无重复元素集合
- map：有序的key/value集合

**4.异常类型：**

- exception：异常类型

**5.服务类型**

- service：具体对应服务的类