[TOC]

# 概述

> Action中三种作用域：request,session,application 对象

Javaweb开发少不了往域对象中存数据，使用sruts2框架该如何往域对象中存数据呢？使用sruts2有三种方式往域对象中存数据，各有各的优点

# 一、调用Servlet API

> 核心类是ServletActionContext提供的静态的方法

## 第一步：引入相关包

## 第二步：配置Struts2的过滤器

> WebRoot/WEB-INF/web.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" id="WebApp_ID" version="3.0">
	<welcome-file-list>
		<welcome-file>index.jsp</welcome-file>
	</welcome-file-list>

	<!-- struts2过滤器 -->
	<filter>
		<!-- 过滤器名称 -->
    	<filter-name>struts2</filter-name>
    	<!-- 过滤器类 -->
    	<filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
  	</filter>
  	<!-- struts2过滤器映射 -->
  	<filter-mapping>
  		<!-- 过滤器名称 -->
    	<filter-name>struts2</filter-name>
    	<!-- 过滤器映射 -->
    	<url-pattern>/*</url-pattern>
  	</filter-mapping>
</web-app>
```

## 第三步：开发Action

> TestAction.java

```java
public String execute() throws Exception {

	HttpServletRequest request = ServletActionContext.getRequest();
	
	ServletContext application = ServletActionContext.getServletContext();
	
	HttpSession session = request.getSession();
	
	//操作
	request.setAttribute("request_data", "this is request");
	application.setAttribute("application_data", "this is application");
	session.setAttribute("session_data", "this is session");

	return SUCCESS;
}
```

## 第四步：Struts2的配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE struts PUBLIC
          "-//Apache Software Foundation//DTD Struts Configuration 2.0//EN"
          "http://struts.apache.org/dtds/struts-2.0.dtd">
<struts>
	<package name="data" namespace="/" extends="struts-default">
		<!-- 全局变量 -->
		<global-results>
			<result name="success">/data.jsp</result>
		</global-results>
		 
		<action name="data" class="com.theliang.d_data.a_DataAction">
			<!-- 返回结果标记success对应的页面，如果在当前action中没有配置，就会去全局跳转视图中查找 -->
		</action>
	</package>    
</struts>
```

# 二、通过ActionContext获取不同的(request/session/application)Map

```java
/**
*	【推荐：解藕的方式实现对数据的操作】
*/
public String execute() throws Exception {

 	//getContext()是一个静态方法，可以直接调用，返回值就是ActionContext
	ActionContext ac=ActionContext.getContext();

	//得到Strust对HttpServletRequest对象进行了封装，封装为了一个Map
	//得到代表request的map集合
	Map<String, Object> request = ac.getContextMap();
	//得到代表session的map集合
	Map<String, Object> session = ac.getSession();
	//得到代表servletContext的map集合
	Map<String, Object> application = ac.getApplication();

	// 向作用域中存储数据
	request.put("request_data","this is request");
	session.put("session_data", "this is session");
	application.put("application_data", "this is application");

	return SUCCESS;
}
```

## ActionContext映射数据

```java
public String execute() throws Exception {
	ActionContext ac=ActionContext.getContext();
	
	//映射数据
	//	Map<String, Object> request=ac.getContextMap();
//	request.put("request_data","request_data");
	ac.getContextMap().put("request_data","request_data_context");//如果request也设置相同的数据则输出的是那个request，不输出这个
	ac.getContextMap().put("cn","China_context");
//	ac.getContextMap().put("request","request_context");//ERROR，键值不能为域对象关键字


	/*******************此request与以上两种方式有区别**************************/
	Map<String, Object> request=(Map<String, Object>) ac.get("request");
	request.put("request_data","request_data");
	request.put("cn","China");
	
	ac.getApplication().put("context_data","context_data");
	ac.getSession().put("session_data","session_data");
	return "success";
}
```


# 三、实现接口的方法：(RequestAware/SessionAware/ApplicationAware)

```java
public class ScopeAction implements RequestAware,SessionAware,ApplicationAware{

    private Map<String,Object> request;
    private Map<String,Object> session;
    private Map<String,Object> application;
    // struts运行时候，会把代表request/session/application的map对象注入
    public void setRequest(Map<String, Object> request) {
        this.request = request;
    }
	public void setSession(Map<String, Object> session) {
        this.session = session;
    }
    public void setApplication(Map<String, Object> application) {
        this.application = application;
    }
    
    public String execute(){
        //map使用put设置值
        request.put("requestKey", "requestValue");
        session.put("sessionKey", "sessionbValue");
        application.put("applicationKey", "applicationValue");
        
        return "success";
    }
}
```

