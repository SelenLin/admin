[TOC]

# 开发步骤

spring各个版本中：

在3.0以下的版本，源码有spring中相关的所有包【spring功能 + 依赖包】

在3.0以上的版本，源码中只有spring的核心功能包【没有依赖包】
		(如果要用依赖包，需要单独下载！)

## 1) 导入源码,jar文件

`spring-framework-3.2.5.RELEASE`

    commons-logging-1.1.3.jar           日志
	spring-beans-3.2.5.RELEASE.jar      bean节点
	spring-context-3.2.5.RELEASE.jar    spring上下文节点
	spring-core-3.2.5.RELEASE.jar       spring核心功能
	spring-expression-3.2.5.RELEASE.jar spring表达式相关表

> 以上是必须引入的5个jar文件，在项目中可以用户库管理！

## 2) 核心配置文件

`applicationContext.xml`

Spring配置文件：`applicationContext.xml / bean.xml`

约束参考：
`spring-framework-3.2.5.RELEASE\docs\spring-framework-reference\htmlsingle\index.html`

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
</beans>   
```

## 3) API

> ApplicationContext.xml 中的配置 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">
	
	<!-- IOC容器的配置： 要创建的所有的对象都配置在这里 -->
	<bean id="user" class="com.theliang.a_hello.User" ></bean>
</beans> 
```

> 创建对象测试类

```java
package com.theliang.a_hello;

import org.junit.Test;
import org.springframework.beans.factory.BeanFactory;
import org.springframework.beans.factory.xml.XmlBeanFactory;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.core.io.ClassPathResource;
import org.springframework.core.io.Resource;

public class App1_get_ioc {
	@Test
	public void testIOC(){
		
		// 直接创建对象
		//User user=new User();
		
		// 现在，把对象的创建交给spring的IOC容器
		Resource resource=new ClassPathResource("com/theliang/a_hello/applicationContext.xml");
		// 创建容器对象(Bean的工厂), IOC容器 = 工厂类 + applicationContext.xml
		BeanFactory factory=new XmlBeanFactory(resource);
		
		User user=(User)factory.getBean("user");
		
		System.out.println(user);
	}
	
	//2. （方便）直接得到IOC容器对象 
	@Test
	public void testAc() throws Exception {
		// 得到IOC容器对象
		ApplicationContext ac = new ClassPathXmlApplicationContext("com/theliang/a_hello/applicationContext.xml");

		// 从容器中获取bean
		User user = (User) ac.getBean("user");
		
		System.out.println(user);
	}
}
```
