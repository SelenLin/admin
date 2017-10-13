[TOC]

# bean对象创建的细节

## 对象创建：单例/多例

* `scope="singleton"` 默认值；即默认是单例【service/dao/工具类】  
* `scope="prototype"` 多例；【Action对象】

> applicationContext.xml 部分配置

```xml
<bean id="user" class="com.theliang.a_hello.User" scope="singleton"></bean>
```

> 实体类 User.java

```java
package com.theliang.a_hello;
public class User {
	private int id;
	private String name;
	public User() {
		super();
		System.out.println("------User对象创建------");
	}
	// set get ...
}
```

> bean测试类

```java
public void testIOC() throws Exception {
	// 得到IOC容器对象  【用实现类，因为要调用销毁的方法】
	ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("com/theliang/a_hello/applicationContext.xml");
	System.out.println("-----容器创建-----");
	
	// 从容器中获取bean
	User user1 = (User) ac.getBean("user");
	User user2 = (User) ac.getBean("user");
	
	System.out.println(user1);
	System.out.println(user2);
}
```

## 什么时候创建？

* `scope="singleton"`  在启动(容器初始化之前)， 就已经创建了bean，且整个应用只有一个  
* `scope="prototype"`  在用到对象的时候，才创建对象  

> 对于`singleton` 输出的结果是：

	------User对象创建------
	-----容器创建-----
	com.theliang.a_hello.User@7b49cea0
	com.theliang.a_hello.User@7b49cea0

> 对于`prototype` 输出的结果是：

	-----容器创建-----
	------User对象创建------
	------User对象创建------
	com.theliang.a_hello.User@6325a3ee
	com.theliang.a_hello.User@1d16f93d

## 是否延迟创建

* lazy-init="false"  默认为false,  不延迟创建，即在启动时候就创建对象  
* lazy-init="true"   延迟初始化， 在用到对象的时候才创建对象（只对单例有效）

> 对于`scope="singleton"` `lazy-init="true"` 输出的结果是：

	-----容器创建-----
	------User对象创建------
	com.theliang.a_hello.User@61f8bee4
	com.theliang.a_hello.User@61f8bee4

## 创建对象之后，初始化/销毁

* `init-method="init_user"`  【对应对象的init_user方法，在对象创建之后执行 】  
* `destroy-method="destroy_user"` 【在调用容器对象的destroy方法时候执行，(容器用实现类)】  

> 实体类 User.java

```java
package com.theliang.a_hello;
public class User {
	private int id;
	private String name;
	public User() {
		super();
		System.out.println("------User对象创建------");
	}
	public void init_user() {
		System.out.println("创建对象之后，初始化");
	}
	public void destroy_user() {
		System.out.println("IOC容器销毁，user对象回收!");
	}
	// set get ...
}
```

> bean测试类

```java
public void testIOC() throws Exception {
	// 得到IOC容器对象  【用实现类，因为要调用销毁的方法】
	ClassPathXmlApplicationContext ac = new ClassPathXmlApplicationContext("com/theliang/a_hello/applicationContext.xml");
	System.out.println("-----容器创建-----");
	
	// 从容器中获取bean
	User user1 = (User) ac.getBean("user");
	User user2 = (User) ac.getBean("user");
	
	System.out.println(user1);
	System.out.println(user2);

	// 销毁容器对象 
	ac.destroy();
}
```

> 输出的结果

	------User对象创建------
	创建对象之后，初始化
	-----容器创建-----
	com.theliang.a_hello.User@7b49cea0
	com.theliang.a_hello.User@7b49cea0
	IOC容器销毁，user对象回收!

> 注意：在bean配置中，如果配置的是`scope="prototype"`，则不调用destroy方法

