---
title: Spring
date: 2022-09-01 12:00:00
categories: ["学习"]
tags: ["复习笔记","Spring"]
mathjax: true
---

# spring



## IOC容器

org.springframework.context.ApplicationContext接口代表IOC容器，负责实例化，配置和组装Bean。

ClassPathXmlApplicationContext通过xml定义配置文件，是ApplicationContext的一种接口实现。

IOC可以帮助实例化对象，这是一个工厂模式。

本质上是一个BeanFactory

### 配置元数据

在xml中的<beans>标签中配置，<bean>中定义id与class类全限定名

<!--more-->

## bean实例化

构造器实例化

（了解）静态工厂实例化

（了解）实例化工厂实例化

IOC容器，容器底层也是一个beanfactory，ApplicationContext会对xml文件进行一个解析，在内部形成一个map映射，并根据配置文件生成对应的bean，通过id从map中获得对应的bean。

默认使用空构造器实例化

容器底层通过java反射机制，通过全限定名拿到对应字节码的类，通过调用类的构造器方法获得object，成功创造一个bean。

## spring IOC 注入

主动实例化与外部引入

外部引入通过带参方法传过来

![image-20220413190029986](spring\image-20220413190029986.png)

注入：把外部创建好的对象，传入到依赖中

### set方法注入

![image-20220413190911173](spring\image-20220413190911173.png)

在xml文件的<bean>标签中加入<property>标签

![image-20220413190850025](spring\image-20220413190850025.png)

name：属性字段名称

ref：对用bean标签的id属性值

value: 具体的值 很少有这种注入，一般都是bean对象注入

### 构造器注入

![image-20220413191655598](spring\image-20220413191655598.png)

![image-20220413191713980](spring\image-20220413191713980.png)

### 循环依赖问题

A依赖于B， B依赖于A

B要注入进A，A要注入进B

spring怎么处理？

首先，如果两个bean用构造器注入则无法实现，因为在构造时还未实例化。解决方法，使用set方法注入。

set方法注入会将实例化却未完成初始化的调用者A先加入到earlySingletonObjects中，之后实例化B的时候就从这个缓存中将未初始化的A注入到B中，之后完成B的初始化，最后完成A的初始化。

## 自动化装配

![image-20220413195210547](spring\image-20220413195210547.png)

@Resource

​	默认根据属性字段名查找bean id，默认成自然，不相等会查找类型

​	自动注入默认使用set注入，属性字段可以不提供set方法	

​	注解可以声明在属性字段级别或set方法级别

​	可以设置注解name属性，若设置name属性，name属性值必须与bean id值一致（接口区分实现类）

@Autowired注解

​	默认根据class注入	

​	可以结合@Qualifier注解寻找id值

## Spring IOC 扫描器

IOC扫描器，用注解代替配置文件，简化开发配置

![image-20220413205644230](spring\image-20220413205644230.png)

xml中要开启自动扫描并定义扫描范围，一般选择根目录。

​	![image-20220413205751721](spring\image-20220413205751721.png)

@Controller

@Service

@Repository

@Component

声明在类级别上，默认id为类的首字母小写。

![image-20220413211126767](spring\image-20220413211126767.png)

## 理解

spring装配的都是功能类，一般都是单例的

pojo与DAO的交互要用到Mybatis

## Bean的作用域与生命周期

### 作用域

Lazy-init只用在程序调用时才会实例化

正常spring容器启动时就会实例化所用Bean



Lazy-init默认false，可以发现潜在配置问题，使用时不用实例化加快效率



不存在会改变对象状态的成员变量才适合单例：controller service dao

单例模式的bean会在实例化后放入缓存池中

原型不会放入缓存池中

### 生命周期

Bean的定义，初始化，使用和销毁4个阶段

#### 定义

配置文档

#### 初始化

实例化对象，可以绑定init-method来查看实例化时间

#### Bean的使用

用Beanfactory getbean得到对象

#### 销毁

ctx close后调用destroy-method进行销毁

## spring task

定时任务

@Scheduled

秒分时日月星期年

* *表示每

## Spring AOP

### 代理模式

为在服务类与客户类之间插入其他功能，插入功能对于调用者透明，起到伪装控制的作用。委托类与代理类有共同的行为，因此他们要有一个共同的父类或父接口。

可以在代理中设置增强行为。

静态代理，比较死板，

### 动态代理

#### jdk动态代理

在程序运行时通过反射机制自动生成代理对象。

目标对象不固定

程序执行时动态创建目标对象

代理对象会增强目标对象的行为

![image-20220414123805849](spring\image-20220414123805849.png)

![image-20220414124047966](spring\image-20220414124047966.png)

invoke就是反射中的机制

用代理对象调用目标对象方法，会直接进入Invocationhandler接口中的invoke函数，在invoke函数中可以对行为进行增强，并用反射对目标对象方法进行invoke。

![image-20220414125155566](spring\image-20220414125155566.png)

![image-20220414125204730](spring\image-20220414125204730.png)

newProxyInstance是一个反射机制，利用loader加载器，创造一个实现interfaces的类，满足对应invoke函数

代理的目标对象必须要有接口实现。



#### CGLIB 动态代理

创建子类，对对应方法进行重写，采用继承实现，不能对final的类进行代理

![image-20220414130004678](spring\image-20220414130004678.png)

setCallBack回调方法拦截器

目标对象

代理类

代理对象

调用方法

### Spring AOP

日志处理，事务处理，权限控制，性能统计

为了防止大段代码相同。若采用封装的方法，还要主动调用对应的方法。强耦合。

代理模式进行解耦

![image-20220414151744818](spring\image-20220414151744818.png)

装饰器模式/代理模式

每个业务类都要有一个代理类

采用AOP

AOP考虑面到面的切入（方法拦截器），实现公共功能的重复使用

#### AOP特点

降低耦合度

提高代码复用性

提高系统扩展性

可以不影响原有功能基础上添加新功能

#### AOP基本概念

Joinpoint 每一个方法代表一个连接点

Pointcut 匹配拦截的规则切入点

Advice 通知增强操作 前置，返回，异常，最终，环绕（需要手动调一下方法）

Aspect 切面 切入点与通知的结合

Target 目标对象

#### 对AOP的理解

AOP是什么？ AOP作用？AOP公共性功能？AOP特点？AOP底层？

## 项目

spring 是基础框架提供IOC容器与AOP面向切面编程

Mybatis 提供数据持久层支持，通过配置文件Mapper.xml对应到Dao层的接口，返回对应的POJO。

SpringMVC目前理解主要提供Restful架构，直接通过URI访问Controller

各种配置都放在java下resources文件夹中作为资源文件夹

#### AOP

规定切入点与Advice，通过注解或XML

@Pointcut

@beforeAdvice

Advice 通知增强操作 前置，返回，异常，最终，环绕（需要手动调一下方法）

@Component 切面类也需要IOC实例化注入

@Aspect 需要声明切面类

## spring 事务

为了让方法满足ACID特性

具体怎么实现是JDBC，Hibernate等平台内部处理的

事务管理就是通过AOP完成的

## spring MVC

### MVC

将业务逻辑从界面中分离出来，允许他们单独改变不互相影响

![image-20220414180208227](spring\image-20220414180208227.png)

### spring MVC

采用请求响应模型，帮助简化开发，降低耦合度

中央控制器DIspatcherServlet，处理器映射器（Handler Mapping），视图解析器（View Resolver），约定大于配置

### 能做什么

简单干净的设计出Web层

约定大于配置

URL到页面控制器的映射

Model方便与其他视图进行集成

更加简单的异常处理

对静态资源的支持

支持Restful风格



### spring MVC请求流程

![image-20220414181013366](spring\image-20220414181013366.png)

适配器是与拦截器对应的

![image-20220414181731549](spring\image-20220414181731549.png)

![image-20220414181745956](D:\课件\CS-Notes\notes\spring\image-20220414181745956.png)

### spring boot

自带服务器，简化spring搭建
