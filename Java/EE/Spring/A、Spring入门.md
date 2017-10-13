[TOC]

# 概念

* Spring是一个非常活跃的开源框架, 它是一个基于IOC和AOP来构架多层JavaEE系统的框架,它的主要目地是简化企业开发

* Spring以一种非侵入式的方式来管理你的代码, Spring提倡”最少侵入”，这也就意味着你可以适当的时候安装或卸载Spring

* Spring框架的主页： http://www.springsource.org/

* Spring框架下载地址：http://www.springsource.org/download

# 为什么要使用Spring 

在项目中引入spring立即可以带来下面的好处

* 降低组件之间的耦合度,实现软件各层之间的解耦

```
graph LR
Controller-->Service
Service-->Dao
```

**Controller-->Service-->Dao**

1. IOC(Inversion of Control，即“控制反转”)：包含并管理应用对象的配置和生命周期，你可以配置你的每个bean如何被创建，也可以配置每个bean是只有一个实例，还是每次需要时都生成一个新的实例，以及它们是如何相互关联的

2. AOP(Aspect Oriented Programming 面向切面编程)：Spring的 AOP 封装包提供了符合AOP Alliance规范的面向方面的编程实现，让你可以定义，例如方法拦截器（method-interceptors）和切点（pointcuts），从逻辑上讲，从而减弱代码的功能耦合，清晰的被分离开。而且，利用source-level的元数据功能，还可以将各种行为信息合并到你的代码中。

3. DAO(Data Access Object):Spring还提供了对数据库JDBC的封装,使用JdbcTemplate来简化数据操作

4. ORM（Object/Relation Mapping）:ORM 封装包提供了常用的“对象/关系”映射APIs的集成层。 其中包括JPA、JDO、Hibernate 和 iBatis。利用ORM封装包，可以混合使用所有Spring提供的特性进行“对象/关系”映射，如前边提到的简单声明性事务管理。

5. JEE: 对Java企业级开发提供了一些解决方案,例如EJB、JMS等

6. WEB: Spring中的 Web 包提供了基础的针对Web开发的集成特性，例如多方文件上传，利用Servlet listeners进行IOC容器初始化和针对Web的ApplicationContext。当与WebWork或Struts一起使用Spring时，这个包使Spring可与其他框架结合。

# 专业术语了解

## 组件/框架设计

### 侵入式设计

引入了框架，对现有的类的结构有影响；即需要实现或继承某些特定类。  

	例如：	Struts框架

### 非侵入式设计

引入了框架，对现有的类结构没有影响。

	例如：Hibernate框架 / Spring框架


## 控制反转

Inversion on Control , 控制反转 IOC
对象的创建交给外部容器完成，这个就做控制反转

##　依赖注入

dependency injection
处理对象的依赖关系

### 区别

控制反转， 解决对象创建的问题 【对象创建交给别人】

依赖注入，在创建完对象后， 对象的关系的处理就是依赖注入 【通过set方法依赖注入】

AOP	面向切面编程。切面，简单来说来可以理解为一个类，由很多重复代码形成的类。
	切面举例：事务、日志、权限;

