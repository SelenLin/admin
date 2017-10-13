[TOC]

#　Struts2与Spring整合

Spring，负责对象对象创建
Struts， 用Action处理请求

Spring与Struts框架整合，
	关键点：让struts框架action对象的创建，交给spring完成！

## 1. 导入相关jar包

### Struts2的jar包

	commons-fileupload-1.2.2.jar
	commons-io-2.0.1.jar
	commons-lang3-3.1.jar
	freemarker-2.3.19.jar
	javassist-3.11.0.GA.jar
	ognl-3.0.5.jar
	struts2-core-2.3.4.1.jar
	xwork-core-2.3.4.1.jar

### Spring-core的jar包

	commons-logging-1.1.3.jar
	spring-beans-4.0.5.RELEASE.jar
	spring-context-4.0.5.RELEASE.jar
	spring-core-4.0.5.RELEASE.jar
	spring-expression-4.0.5.RELEASE.jar

### Spring-web 支持jar包

	spring-web-3.2.0.RELEASE.jar
	struts2-spring-plugin-2.3.4.1.jar

## 2. 相关配置

### web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" id="WebApp_ID" version="3.0">

<!-- 1. struts配置 -->
	<filter>
		<filter-name>struts2</filter-name>
		<filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
	</filter>
	<filter-mapping>
		<filter-name>struts2</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>
	
<!-- 2. spring 配置 -->
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>/WEB-INF/classes/bean-*.xml</param-value>
	</context-param>
	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>
</web-app>
```


### struts.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE struts PUBLIC
	"-//Apache Software Foundation//DTD Struts Configuration 2.3//EN"
	"http://struts.apache.org/dtds/struts-2.3.dtd">
<struts>

	<package name="user" extends="struts-default">
		<action name="user" class="userAction" method="execute">
			<result name="success">/index.jsp</result>
		</action>
	</package>
</struts>
```

### bean.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
	xmlns:p="http://www.springframework.org/schema/p"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
    	 http://www.springframework.org/schema/beans/spring-beans.xsd
     	 http://www.springframework.org/schema/context
         http://www.springframework.org/schema/context/spring-context.xsd
         http://www.springframework.org/schema/aop
         http://www.springframework.org/schema/aop/spring-aop.xsd
         http://www.springframework.org/schema/tx
     	 http://www.springframework.org/schema/tx/spring-tx.xsd">

    <!-- 指定dao单例 -->
	<bean id="userDao" class="com.theliang.dao.UserDao" scope="singleton" lazy-init="false"></bean>

	<bean id="userService" class="com.theliang.service.UserService">
		<property name="userDao" ref="userDao"></property>
	</bean>

	<!-- 指定action多例 -->
	<bean id="userAction" class="com.theliang.action.UserAction" scope="prototype">
		<property name="userService" ref="userService"></property>
	</bean>

	<!-- 可以将dao service action分成三个配置文件，更利于维护，写法不变 -->
</beans>
```

## 3. java类

### UserDao.java

```java
package com.theliang.dao;
public class UserDao {
	public void save() {
		System.out.println("DB:保存用户");
	}
}
```

### UserService.java

```java
package com.theliang.service;
import com.theliang.dao.UserDao;
public class UserService {
	//IOC容器注入
	private UserDao userDao;
	public void setUserDao(UserDao userDao) {
		this.userDao = userDao;
	}
	public void save() {
		userDao.save();
	}
}
```

### UserAction.java

```java
package com.theliang.action;
import com.opensymphony.xwork2.ActionSupport;
import com.theliang.service.UserService;
public class UserAction extends ActionSupport {
	// springIOC容器注入
	private UserService userService;
	public void setUserService(UserService userService) {
		this.userService = userService;
	}
	public String execute() {
		userService.save();
		return SUCCESS;
	}
}
```

## 4. 其它

### jsp页面

```html
<%@ page language="java" import="java.util.*" pageEncoding="utf-8"%>
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
  <head> 
    <title>Struts2+Spring</title>
  </head>
  <body>
    Spring与Struts2整合完成！
  </body>
</html>
```