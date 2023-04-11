---
title: 计算机系统大作业——Hello的一生
date: 2020-08-04 12:42:24
tags: ['计算机系统','P2P','程序运行全过程']
categories: ['学习']
---

# 计算机系统大作业——Hello的一生



**摘 要**

  本文以hello.c为例，探寻程序由代码一步步经过预处理、编译、链接、之后成为进程，并被系统进行管理的过程。从而窥探程序由出生到死去的过程中，计算机系统各个部分是如何协调工作的，随着hello.c的一步步变化逐渐理解机器的运行机制。

**关键词：**计算机系统；P2P；程序运行全过程；             

<!-- more -->





# 第1章 概述

## 1.1 Hello简介

hello.c先经过预处理变为hello.i，后经过编译变为汇编文件hello.s，再经过汇编变为可重定向目标文件hello.o，最后经过链接变为可执行目标文件hello.out

在bash中运行hello，运行时bash先fork一个新进程，之后用execve加载执行hello，bash在用户模式下运行hello，输出完毕后Hello进程结束变为僵死进程，由bash回收。

## 1.2 环境与工具 

### 1.2.1 硬件环境 

X64 CPU；2GHz；2G RAM；256GHD Disk 以上

### 1.2.2 软件环境 

Windows10 64位; VMWARE14; Ubuntu 16.04

### 1.2.3 开发工具

Vmware 14；Ubuntu 16.04 LTS 64 位；CodeBlocks；vi/ gpedit+gcc，EDB，VScode



## 1.3 中间结果

| hello.c   | 源c文件                  |
| --------- | ------------------------ |
| hello.i   | 预处理后的c源文件        |
| hello.s   | 编译后的汇编文件         |
| hello.o   | 汇编后的可重定位目标文件 |
| hello.out | 可执行程序               |

## 1.4 本章小结

本章简要说明hello的P2P 020 过程，列出开发环境及论文过程中的中间结果。







# 第2章 预处理

## 2.1 预处理的概念与作用 

预处理是指在程序源代码被翻译为目标代码的过程中，在编译之前的工作，对一个源文件进行编译时， 系统将自动引用预处理程序对源程序中的预处理部分作处理， 处理完毕自动进入对源程序的编译。

Ｃ语言提供了多种预处理功能，如宏定义、文件包含、 条件编译等。

\#if/#ifdef/#ifndef/#else/#elif/#endif（条件编译）、#define（宏定义）、#include（源文件包含）、#line（行控制）、#error（错误指令）、#pragma（和实现相关的杂注）以及单独的#（空指令）



## 2.2在Ubuntu下预处理的命令 

gcc -E hello.c -o hello.i

   

​    ![img](https://img-blog.csdnimg.cn/20181231223233808.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNzQ3NzQz,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![img](https://img-blog.csdnimg.cn/20181231223233806.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

   



## 2.3 Hello的预处理结果解析 

Hello.c经预处理后变为hello.i ,实际是一个搜罗各家武功最后另成一派的过程。hello.c中的预处理指令是

\#include <stdio.h>

\#include <unistd.h>

\#include <stdlib.h>

而之后程序使用的函数，均是这三个库里的内容，预处理就是把程序具体引用库函数的代码与c文件合在一起。



## 2.4 本章小结

预处理在编译前开始，这个过程并不对程序的源代码进行解析，但它把源代码分割或处理成为特定的单位——预处理记号。

c语言的预处理会展开以#起始的行，试图解释为预处理指令，预处理指令一般被用来使源代码在不同的执行环境中被方便的修改或者编译。







# 第3章 编译 

## 3.1 编译的概念与作用

编译(compilation , compile) 1、利用编译程序从源语言编写的源程序产生目标程序的过程。 2、用编译程序产生目标程序的动作。 编译就是把高级语言变成计算机可以识别的2进制语言，计算机只认识1和0，编译程序把人们熟悉的语言换成2进制的。 编译程序把一个源程序翻译成目标程序的工作过程分为五个阶段：词法分析；语法分析；语义检查和中间代码生成；代码优化；目标代码生成。主要是进行词法分析和语法分析，又称为源程序分析，分析过程中发现有语法错误，给出提示信息。（但此处编译单指由预处理后文件到汇编文件的过程。）

把预处理完的文件进行一系列语法分析及优化后生成相应的汇编文件

## 3.2 在Ubuntu下编译的命令 

gcc -S hello.i -o hello.s

![img](https://img-blog.csdnimg.cn/20181231223233852.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNzQ3NzQz,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![img](https://img-blog.csdnimg.cn/20181231223233860.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## 3.3 Hello的编译结果解析 

\3. 3. 1 数据类型

 1.整型数据

 ![img](https://img-blog.csdnimg.cn/20181231223233886.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

 Sleepsecs整型全局变量，占四个字节，.long 声明一个32位的数，相当于赋值操作，



 ![img](https://img-blog.csdnimg.cn/20181231223234210.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

 对sleepsecs取值



​      ![img](https://img-blog.csdnimg.cn/20181231223234208.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

​    Main函数的参数之一，命令行个数int argc



​    ![img](https://img-blog.csdnimg.cn/20181231223234272.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

​    i为循环辅助变量，int i的位置



   2.字符串

   ![img](https://img-blog.csdnimg.cn/20181231223234290.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

   两个字符串类型均在只读段中，属于字符串常量



   3.字符串指针

![img](https://img-blog.csdnimg.cn/20181231223234304.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



\3. 3. 2 操作



   1.算术操作：

   ![img](https://img-blog.csdnimg.cn/20181231223234322.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

   让循环控制变量加一





   2.赋值操作：

​        ![img](https://img-blog.csdnimg.cn/20181231223234319.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

   为循环控制变量i赋初值0





   3.关系操作

​        ![img](https://img-blog.csdnimg.cn/20181231223234377.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

   判断是否argc！=3



​        ![img](https://img-blog.csdnimg.cn/20181231223234372.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

   判断是否i<10



 4.数组操作

   ![img](https://img-blog.csdnimg.cn/20181231223234701.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

   从数组中取值，数组存储在栈中，利用%rbp来访问数组中的值



\3. 3. 3控制转移

​        ![img](https://img-blog.csdnimg.cn/20181231223234703.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

   If（argc！=3）输出字符串后退出。否则进入循环

   

   ![img](https://img-blog.csdnimg.cn/20181231223234699.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

   为循环控制变量i赋初值0

   ![img](https://img-blog.csdnimg.cn/20181231223234694.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

   跳出循环的条件判断 若i<=9则继续进入循环内容（L4）

   ![img](https://img-blog.csdnimg.cn/20181231223234724.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNzQ3NzQz,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

   L4完成后i++

   完成for(i=0;i<10;i++)的架构



\3. 3. 4函数操作

   ![img](https://img-blog.csdnimg.cn/20181231223234832.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNzQ3NzQz,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

   通过call指令来调用函数

   ![img](https://img-blog.csdnimg.cn/20181231223234829.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

   ret指令使函数返回 返回值存储在%rax中

## 3.4 本章小结

   本章将经过预处理的.i文件编译成汇编文件.s，体现了由高级语言到汇编语言的转换过程，说明了其中的各种操作和数据是怎样在汇编语言中实现的。汇编是由高级语言到机器语言必不可少的一环，理解汇编语言的架构，是对机器的又一层理解，对今后的调试与优化工作有很大益处。





# 第4章 汇编 

## 4.1 汇编的概念与作用

​    由汇编器将汇编文件翻译成机器指令，并将这些机器指令打包为可重定位目标程序





## 4.2 在Ubuntu下汇编的命令

​    ![img](https://img-blog.csdnimg.cn/2018123122323514.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![img](https://img-blog.csdnimg.cn/20181231223235156.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



## 4.3 可重定位目标elf格式

![img](https://img-blog.csdnimg.cn/20181231223236514.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNzQ3NzQz,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

Elf头记录了魔数、文件类型、系统架构等信息。

![img](https://img-blog.csdnimg.cn/20181231223245202.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNzQ3NzQz,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

节头表通过数组实现，每个数组项包含一个节的信息。各个节构成了程序头表中定义的各段的内容。

.rela.text的link与info分别是符号表与代码段，表示将利用符号表在重定位时修改代码段

![img](https://img-blog.csdnimg.cn/20181231223238848.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

因全局变量定要经过重定义，这里以全局变量sleepsecs为例。在.rela.text中sleepsecs的offset是0x5c，表示需要修改的在.text段的0x5c处，在之后链接时要将sleepsecs链接到这里，而sleepsecs在符号表中的位置为0x9，重定位的类型为0x2。

![img](https://img-blog.csdnimg.cn/20181231223238981.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNzQ3NzQz,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

Symbol存放着所有全局符号

## 4.4 Hello.o的结果解析

![img](https://img-blog.csdnimg.cn/20181231223249348.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNzQ3NzQz,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

总体来讲反汇编结果和第三章的汇编代码差别不大，主要差别为

1. 函数调用方面：汇编语言中call指令后直接跟函数名称，而反汇编后的代码call后是代码行索引。
2. 分支转移方面：汇编语言中jmp指令后直接跟标签名称，而反汇编后的代码jmp后是代码行索引。
3. 反汇编后得到的代码中，每一个需要重定位的符号都以类型+名称形式显示出来。



## 4.5 本章小结

本章表现了汇编语言转化为机器语言并被打包为可重定位目标程序的过程，揭示了机器代码与汇编语言的联系与映射，处理重定位信息的方法。为下一步链接做好准备。





# 第5章 链接

## 5.1 链接的概念与作用

​    每个模块都有自己的代码、数据（初始化全局变量、未初始化全 局变量、静态变量、局部变量），链接便是合并可重定位目标文件相同的节，来让程序变为一个较小的源文件的集合。

## 5.2 在Ubuntu下链接的命令

![img](https://img-blog.csdnimg.cn/20181231223239218.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNzQ3NzQz,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)![img](https://img-blog.csdnimg.cn/20181231223239292.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)



## 5.3 可执行目标文件hello的格式

分析hello的ELF格式，用readelf等列出其各段的基本信息，包括各段的起始地址，大小等信息。

![img](https://img-blog.csdnimg.cn/20181231223239672.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNzQ3NzQz,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

![img](https://img-blog.csdnimg.cn/20181231223239729.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNzQ3NzQz,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

![img](https://img-blog.csdnimg.cn/20181231223240186.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

段头表标志每段的起始地址与大小

![img](https://img-blog.csdnimg.cn/20181231223241230.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNzQ3NzQz,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## 5.4 hello的虚拟地址空间

  ![img](https://img-blog.csdnimg.cn/20181231223245313.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNzQ3NzQz,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

.plt段，PLT（Procedure Linkage Table）表每一项都是一小段代码，对应于本运行模块要引用的一个全局函数。

![img](https://img-blog.csdnimg.cn/2018123122324460.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNzQ3NzQz,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

.rodata 只读数据段

![img](https://img-blog.csdnimg.cn/20181231223251786.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNzQ3NzQz,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

.text 代码段



## 5.5 链接的重定位过程分析

对hello.o进行反汇编只有.text中main函数的汇编值与在特定位置标记的待重定位符号。而对hello.out进行反汇编发现是以.init与.plt与.text为结构的两段。

hello在进行重定位时参照.rela.text与.rela.data的表格数据，在.symtab中查找到相应的符号，插入到表中记录的段内索引中，其中共享库中的符号会创建.got与.got.plt项来记录重定位信息。



## 5.6 hello的执行流程

| _dl_start | 0x00007f24:0ec259b0 |
| --------- | ------------------- |
| _dl_init  | 0x00007fa9:4304a740 |
| _start    | 0x4004c0            |
| init      | 0x400428            |
| main      | 0x400510            |
| _dl_fini  | 0x00007f24:0ec4a750 |



## 5.7 Hello的动态链接分析

  （*以下格式自行编排，编辑时删除*）

分析hello程序的动态链接项目，通过edb调试，分析在dl_init前后，这些项目的内容变化。要截图标识说明。

![img](https://img-blog.csdnimg.cn/20181231223242606.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNzQ3NzQz,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

dl_init前程序的.plt表

![img](https://img-blog.csdnimg.cn/20181231223244246.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

dl_init前程序的.got表

## 5.8 本章小结

链接分为静态链接与动态链接，静态链接将目标文件和库文件打包至一个可执行文件中，动态链接在可执行目标文件中添加重定向记录，之后通过got表项和延迟绑定的方法实现对符号的重定向。





# 第6章 hello进程管理

## 6.1 进程的概念与作用

​    进程指程序的一次运行过程。更确切说，进程是具有独立功能的一个程序关于某个数据集合的一次运行活动，因而进程具有动态含义。同一个程序处理不同的数据就是不同的进程。进程有自己的生命周期。

​    操作系统以外的都属于“用户”的任务。

​    计算机处理的所有“用户”的任务由进程完成。

​    为强调进程完成的是用户的任务，通常将进程称为用户进程。

​    计算机系统中的任务通常就是指进程。

## 6.2 简述壳Shell-bash的作用与处理流程

shell 是一个交互型应用级程序，代表用户运行其他程序，shell 执行一系列的读/ 求值步骤：读步骤读取用户的命令行；求值步骤解析命令，代表用户运行



## 6.3 Hello的fork进程创建过程

​    用户在shell中输入./hello 学号 姓名 ，shell开始解析命令，先判断./hello是不是内置命令，发现不是后寻找当前目录中是否有名为hello的可执行文件。找到该文件后shell用fork（）创建一个子进程，更改该进程的进程组编号。

## 6.4 Hello的execve过程

​    在新进程中，使用execve（）加载并执行hello，将用户输入的参数列表与环境变量一同加载，之后开始运行hello的main函数中的输出。

## 6.5 Hello的进程执行

Hello进程在创建后运行在用户模式，而hello的上下文一直保存在内核中，进程上下文实际上是进程执行活动全过程的静态描述。我们把已执行过的进程指令和数据在相关寄存器与堆栈中的内容称为上文，把正在执行的指令和数据在寄存器和堆栈中的内容称为正文，把待执行的指令和数据在寄存器与堆栈中的内容称为下文。具体的说，进程上下文包括计算机系统中与执行该进程有关的各种寄存器（例如通用寄存器，程序计数器PC，程序状态字寄存器PS等）的值，程序段在经过编译过后形成的机器指令代码集，数据集及各种堆栈值PCB结构。这里，有关寄存器和栈区的内容是重要的。当系统调用和异常处理出现时，hello进程会陷入内核空间。此时，我们称内核“代表进程执行”并处于进程上下文。除非在此间隙有更高优先级的进程需要执行并由调度器做出了相应调整，否则在内核退出的时候，程序恢复在用户空间继续执行。

进程在大多数情况下是并行的，并行的进程逻辑流在时间上重叠，形成并发流。因此处理器会让并发的各个进程轮流进行，这就是多任务，一个进程只执行控制流的一部分所需的时间段就是一个时间片。一个进程与其他进程一起工作的多任务模式就是时间分片。

![img](https://img-blog.csdnimg.cn/20181231223246935.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNzQ3NzQz,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

在程序执行printf时引发系统调用，hello进程陷入内核运行，当printf函数结束后在将hello进程返回到用户模式中。当程序运行sleep或某个进程要抢占当前进程执行时就要进行上下文切换。首先保存当前进程的上下文，之后恢复新进程的上下文，最后内核将控制传递给新进程，这就是一个调度的过程。



## 6.6 hello的异常与信号处理

（*以下格式自行编排，编辑时删除*）

 hello执行过程中会出现哪几类异常，会产生哪些信号，又怎么处理的。

 程序运行过程中可以按键盘，如不停乱按，包括回车，Ctrl-Z，Ctrl-C等，Ctrl-z后可以运行ps jobs pstree fg kill 等命令，请分别给出各命令及运行结截屏，说明异常与信号的处理。

![img](https://img-blog.csdnimg.cn/20181231223241592.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

输入Ctrl+c后shell发出一个SIGINT中断信号，hello在接受到信号后终止。

![img](https://img-blog.csdnimg.cn/20181231223243542.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

输入Ctrl+z后shell发出一个SIGSTP停止信号，hello在接受到信号后暂停。

![img](https://img-blog.csdnimg.cn/20181231223245259.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNzQ3NzQz,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

进程停止后用kill -9 给hello进程发送一个SIGKILL终止信号，hello进程终止。

## 6.7本章小结

​    本章总结了进程在shell中的工作流程，简单介绍shell执行进程，调度进程，异常处理的过程。还具体举了几个信号体现shell对于异常的处理方法。





# 第7章 hello的存储管理

## 7.1 hello的存储器地址空间

逻辑地址：

1、在有地址变换功能的计算机中,访问指令给出的地址 (操作数) 叫逻辑地址,也叫相对地址。要经过寻址方式的计算或变换才得到内存储器中的物理地址。

2、把用户程序中使用的地址称为相对地址即逻辑地址。

3、逻辑地址由两个16位的地址分量构成，一个为段基值，另一个为偏移量。两个分量均为无符号数编码。

地址空间: 非负整数地址的有序集合 {0, 1, 2, 3 … } 如果地址空间中的整数是连续的，称为线性地址空间

虚拟地址空间: N = 2n 个虚拟地址的集合 {0, 1, 2, 3, …, N-1}

物理地址空间: M = 2m 个物理内存地址的集合 {0, 1, 2, 3, …, M-1}

## 7.2 Intel逻辑地址到线性地址的变换-段式管理

![https://p-blog.csdn.net/images/p_blog_csdn_net/do2jiang/EntryImages/20090902/3.jpg](https://imgconvert.csdnimg.cn/aHR0cHM6Ly9wLWJsb2cuY3Nkbi5uZXQvaW1hZ2VzL3BfYmxvZ19jc2RuX25ldC9kbzJqaWFuZy9FbnRyeUltYWdlcy8yMDA5MDkwMi8zLmpwZw?x-oss-process=image/format,png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

首先，给定一个完整的逻辑地址[段选择符：段内偏移地址]，



1、看段选择符的T1=0还是1，知道当前要转换是GDT中的段，还是LDT中的段，再根据相应寄存器，得到其地址和大小。我们就有了一个数组了。



2、拿出段选择符中前13位，可以在这个数组中，查找到对应的段描述符，这样，它了Base，即基地址就知道了。



3、把Base + offset，就是要转换的线性地址了。 还是挺简单的，对于软件来讲，原则上就需要把硬件转换所需的信息准备好，就可以让硬件来完成这个转换了。



## 7.3 Hello的线性地址到物理地址的变换-页式管理

![img](https://img-blog.csdnimg.cn/20181231223244404.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNzQ3NzQz,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

   分页单元中，页目录是唯一的，它的地址放在CPU的cr3寄存器中，是进行地址转换的开始点。



   每一个活动的进程，因为都有其独立的对应的虚似内存（页目录也是唯一的），那么它也对应了一个独立的页目录地址。——运行一个进程，需要将它的页目录地址放到cr3寄存器中，将别个的保存下来。



   每一个32位的线性地址被划分为三部份，面目录索引(10位)：页表索引(10位)：偏移(12位) 依据以下步骤进行转换：



(1)   从cr3中取出进程的页目录地址（操作系统负责在调度进程的时候，把这个地址装入对应寄存器）；



(2)   根据线性地址前十位，在数组中，找到对应的索引项，因为引入了二级管理模式，页目录中的项，不再是页的地址，而是一个页表的地址。（又引入了一个数组），页的地址被放到页表中去了。



(3)   根据线性地址的中间十位，在页表（也是数组）中找到页的起始地址；



(4)   将页的起始地址与线性地址中最后12位相加

## 7.4 TLB与四级页表支持下的VA到PA的变换

![img](https://img-blog.csdnimg.cn/20181231223247833.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNzQ3NzQz,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

VA由VPN、VPO两部分构成，VPN用于查找TLB中的页，VPN被分为TLBI与TLBT，TLBI为快表中组索引，TLBT为标志，快表命中后将PPN与VPO组合获得PA，否则从慢表中寻找PPN再组合成为PA。在四级页表中VPN被分为四份，前三个页表存的都是下一级页表的地址，最后一级页表才是PPN。

## 7.5 三级Cache支持下的物理内存访问



![img](https://img-blog.csdnimg.cn/20181231223247803.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNzQ3NzQz,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

MMU先从L1中根据PA的偏移、索引、标记找寻数据，若不命中，再进入L2，L3，查找，都不命中就从主存中读取数据。

## 7.6 hello进程fork时的内存映射

 为新进程创建虚拟内存 

创建当前进程的的mm_struct, vm_area_struct和页表的原样副 本. 

两个进程中的每个页面都标记为只读，

两个进程中的每个区域结构（vm_area_struct）都标记为私有的 写时复制（COW）

 在新进程中返回时，新进程拥有与调用fork进程相同的虚拟 内存

 随后的写操作通过写时复制机制创建新页面

## 7.7 hello进程execve时的内存映射

execve函数在当前进程 中加载并运行程序 hello.out的步骤:

      删除已存在的用户区域

      创建新的区域结构 

代码和初始化数据映射 到.text和.data区（目标 文件提供） 

.bss和栈映射到匿名文 件

      设置PC，指向代码区域 的入口点 

Linux根据需要换入代码 和数据页面

![img](https://img-blog.csdnimg.cn/20181231223251970.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNzQ3NzQz,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## 7.8 缺页故障与缺页中断处理



缺页故障会产生一个缺页中断，这是缺页异常处理程序会选择页表中的一个页并将它牺牲掉，从磁盘中读取缺失的页放到内存的页表中。

## 7.9动态存储分配管理

​    在程序运行时程序员使用 动态内存分配器 (比如 malloc) 获得虚拟内存.动态内存分配器维护着一个进程的虚拟内存区域，称为堆

​    分配器将堆视为一组不同大小的块 (blocks)的集合来维护，每个块要么是已分配的，要么是空闲的。 

分配器的类型 

显式分配器 : 要求应用显式地释放任何已分配的块  例如，C语言中的 malloc 和 free 

隐式分配器 : 应用检测到已分配块不再被程序所使用，就释放 这个块  比如Java，ML和Lisp等高级语言中的垃圾收集 (garbage collection)

​    动态内存分配器在分配时采取记录空闲块的方法

​    方法 1: 隐式空闲链表 (Implicit list) 通过头部中的大小字 段—隐含地连接所有块

​    ![img](https://img-blog.csdnimg.cn/20181231223241230.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

  方法 2: 显式空闲链表 (Explicit list) 在空闲块中使用指针

![img](https://img-blog.csdnimg.cn/20181231223241583.png)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

  方法 3: 分离的空闲列表 (Segregated free list)  按照大小分类，构成不同大小的空闲链表

  方法 4: 块按大小排序  在每个空闲块中使用一个带指针的平衡树，并使用长度作为权值

## 7.10本章小结

​    本章介绍了利用虚拟内存空间访存的过程，由页到块，由虚拟地址到物理地址。虚拟地址更为进程的创建与加载执行提供了极大的便利。动态存储分配在程序运行的过程中十分必要，不同的分配策略也会影响分配的时间空间效率。





# 第8章 hello的IO管理

## 8.1 Linux的IO设备管理方法

在分时并行多任务系统中，为了合理利用系统设备，达到一定的目标，不允许进程自行决定设备的使用，而是由系统按一定原则统一分配、管理。进程要进行IO操作时，需向操作系统提出IO请求，然后由操作系统根据系统当前的设备使用状况，按照一定的策略，决定对改进程的设备分配。设备的应用领域不同，其物理特性各异，但某些设备之间具有共性，为了简化对设备的管理，可对设备分类，或对同类设备采用相同的管理策略，比如Linux主要将外部IO设备分为字符设备和块设备（又被称为主设备），而同类设备又可能同时存在多个，故而要定位具体设备还需提供“次设备号”。



根据主设备号+次设备号可以去相应的设备开关表中定位具体的设备驱动程序。 内核和设备驱动程序的接口是块设备开关表和字符设备开关表。每一种设备类型在表中占用一个表项，每个表项含有若干数据项，其中有一项为该类设备驱动程序入口地址，在系统调用时引导核心转向适当的驱动程序接口。



Linux系统为各类设备分别配置不同的驱动程序，在用户程序中通过文件操作方式使用设备，如open\close\read\write等，由文件系统根据用户程序指令转向调用具体的设备驱动程序。



对设备特殊文件的系统调用，根据文件类型转入块设备开关表或字符设备开关表进行打开、关闭设备的操作，字符设备特殊文件的系统调用read、write转向字符设备开关表中指示的设备驱动程序，而对普通文件或目录文件的read\write系统调用则通过高速缓冲模块（缓冲区）转向设备驱动模块中的strategy过程。Linux中关于不同设备种类的管理架构如下

![img](https://img-blog.csdnimg.cn/20181231223247875.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxNzQ3NzQz,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

## 8.2 简述Unix IO接口及其函数

open函数 函数原型 int open(char * path,int oflag,...)     

返回值是一个文件描述符  path顾名思义就是文件名 oflage文件是打开方式  第三个形参应用于创建文件时使用 /*创建文件其实还有一个create函数使用 以及openat由于还未使用过这个函数和open的差异 所以不在此处累赘*/     open函数 使用if 判断的时候 注意小细节



read函数  函数原型  ssize_t  read(int fd ,  void* buf , size_t nbytes)  

返回值是文件读取字节数 在好几种情况下会出现返回值不等于文件读取字节数 也就是第三个参数nbytes的情况 第二个形参buf读取到buf的内存 文件偏移量(current file offset)受改变



write函数  函数原型 ssize_t write(int fd , const void* buf, size_t nbytes) 

返回值是文件写入字节数  fd是文件描述符 将buf内容写入nbytes个字节到文件 但这里需要注意默认情况是需要在系统队列中等待写入（打开方式不同也会不同）  //以上三个出错都返回-1



lseek函数off_t lseek(int fd, off_t offset , int whence) 

返回值成功函数返回新的文件偏移量 失败-1  fd文件描述符  off_t是有符号的整数 whence其实是和off_t配套使用的 SEEK_SET文件开始处  SEEK_CUR当前值的相对位置 SEEK_END文件长度+-



close函数原型 int close(int fd)  

返回值 成功返回0 失败—1  关闭文件描述符

## 8.3 printf的实现分析

从vsprintf生成显示信息，到write系统函数，到陷阱-系统调用 int 0x80或syscall.

字符显示驱动子程序：从ASCII到字模库到显示vram（存储每一个点的RGB颜色信息）。

显示芯片按照刷新频率逐行读取vram，并通过信号线向液晶显示器传输每一个点（RGB分量）。

int printf(const char *fmt, ...)

{

int i;

char buf[256];

va_list arg = (va_list)((char*)(&fmt) + 4);

i = vsprintf(buf, fmt, arg);

write(buf, i);

return i;

} 

vsprintf的作用就是格式化。它接受确定输出格式的格式字符串fmt。用格式字符串对个数变化的参数进行格式化，产生格式化输出。 va_list_arg是边长参数列表中的第一个参数的地址。

Write如下

​    write:

   mov eax, _NR_write

   mov ebx, [esp + 4]

   mov ecx, [esp + 8]

   int INT_VECTOR_SYS_CALL

之后调用sys_call输出

## 8.4 getchar的实现分析

异步异常-键盘中断的处理：键盘中断处理子程序。接受按键扫描码转成ascii码，保存到系统的键盘缓冲区。

getchar等调用read系统函数，通过系统调用读取按键ascii码，直到接受到回车键才返回。

getchar有一个int型的返回值.当程序调用getchar时.程序就等着用户按键.用户输入的字符被存放在键盘缓冲区中.直到用户按回车为止(回车字符也放在缓冲区中).当用户键入回车之后,getchar才开始从stdio流中每次读入一个字符.getchar函数的返回值是用户输入的第一个字符的ASCII码,如出错返回-1,且将用户输入的字符回显到屏幕.如用户在按回车之前输入了不止一个字符,其他字符会保留在键盘缓存区中,等待后续getchar调用读取.也就是说,后续的getchar调用不会等待用户按键,而直接读取缓冲区中的字符,直到缓冲区中的字符读完为后,才等待用户按键.

getch与getchar基本功能相同,差别是getch直接从键盘获取键值,不等待用户按回车,只要用户按一个键,getch就立刻返回, getch返回值是用户输入的ASCII码,出错返回-1.输入的字符不会回显在屏幕上.getch函数常用于程序调试中,在调试时,在关键位置显示有关的结果以待查看,然后用getch函数暂停程序运行,当按任意键后程序继续运行.

## 8.5本章小结

​    Linux通过UnixI/O函数来实现主存与IO设备的数据转换。Linux系统使用UnixIO接口将IO设备看做文件进行访问与输出，来实现IO管理



# 结论

1. hello.c先经过预处理变为hello.i
2. hello.i经过编译变为汇编文件hello.s
3. hello.s经过汇编变为可重定向目标文件hello.o
4. hello.o经过链接变为可执行目标文件hello.out
5. 运行时bash先fork一个新进程，之后用execve加载执行hello，bash在用户模式下运行hello
6. Hello与其他进程并行执行，在被抢占或系统调用或异常情况后进入内核模式，之后在再回到用户模式，期间发生上下文切换。
7. Hello的printf与getchar函数由linux的IO管理得以与IO设备进行输入输出
8. Hello进程结束变为僵死进程，由bash回收

一个简单的hello.c，想要需要计算机系统的诸多底层工作，系统各个组成部分的互相协调就像庞大机器的齿轮一样紧密。是以以后在编写或运行程序时，若能与机器相互理解，一定能事半功倍。





# 附件

列出所有的中间产物的文件名，并予以说明起作用。

| hello.c   | 源c文件                  |
| --------- | ------------------------ |
| hello.i   | 预处理后的c源文件        |
| hello.s   | 编译后的汇编文件         |
| hello.o   | 汇编后的可重定位目标文件 |
| hello.out | 可执行程序               |







# 参考文献



[1] https://blog.csdn.net/roger_ranger/article/details/78473886 Linux内核：IO设备的抽象管理方式

[2]  https://www.cnblogs.com/chentest/p/5448483.html 关于unix系统接口 普通文件io的小结

[3] https://www.cnblogs.com/pianist/p/3315801.html  [转]printf 函数实现的深入剖析