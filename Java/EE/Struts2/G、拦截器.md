[TOC]

# 自定义拦截器

## API

	|-- Interceptor  拦截器接口
	   |-- AbstractInterceptor  拦截器默认实现的抽象类； 一般用户只需要继承此类即可继续拦截器开发

	|-- ActionInvocation 拦截器的执行状态，调用下一个拦截器或Action

## 执行流程

![image](http://note.youdao.com/yws/api/personal/file/0CDDAA4689E341FE981C9412042D0340?method=download&shareKey=090caa6fb145a9d87de4f69552c8c6aa)

---

```sequence
title:拦截器执行流程
participant 用户
participant Tomcat服务器
participant 拦截器1
participant 拦截器2
participant Action实例

Tomcat服务器->Tomcat服务器:1.启动：执行所有拦截器创建
Tomcat服务器->拦截器1:2.init()：执行所有初始化方法
Tomcat服务器->拦截器2:3.init()：执行所有初始化方法
用户->Tomcat服务器:4.用户访问
Tomcat服务器->Action实例:5.创建Action实例
Tomcat服务器->拦截器1:6.拦截器interceptor(..)方法
拦截器1->拦截器2:7.invoke();调用下一个拦截器
拦截器2->Action实例:8.invoke();
Action实例->Action实例:9.execute()
Action实例-->拦截器2:10；
拦截器2-->拦截器1:11；
拦截器1-->Tomcat服务器:12；
Tomcat服务器-->用户:13.由服务器响应浏览器
```


> 自定义拦截器要实现`com.opensymphony.xwork2.interceptor.Interceptor`接口

```java
package com.theliang.interceptor;

import com.opensymphony.xwork2.ActionInvocation;
import com.opensymphony.xwork2.interceptor.Interceptor;
/**
 * 自定义拦截器
 * @author Administrator
 *
 */
public class HelloInterceptor implements Interceptor{

	// 启动时候执行
	public HelloInterceptor() {
		System.out.println("创建了拦截器对象");
	}

	// 启动时候执行
	@Override
	public void init() {
		System.out.println("执行了拦截器的初始化方法");
	}
	
	// 拦截器业务处理方法 （在访问action时候执行？ 在execute之前执行？）
	@Override
	public String intercept(ActionInvocation invocation) throws Exception {
		System.out.println("2. 执行Action之前");
		// 调用下一个拦截器或执行Action  (相当于chain.doFilter(..)
		
		// 获取的是： execute方法的返回值
		String resultFlag=invocation.invoke();
		
		System.out.println("4. 拦截器，业务处理-结束" + resultFlag);
		
		return resultFlag;
	}
	@Override
	public void destroy() {
		System.out.println("销毁....");
	}
}
```

> 这是一个Action类对象

```java
package com.theliang.g_interceptor;
import com.opensymphony.xwork2.ActionSupport;
/**
 * Action开发测试
 */
public class HelloAction extends ActionSupport{
	public HelloAction(){
		System.out.println("1.Action实例创建了");
	}
	@Override
	public String execute() throws Exception {
		System.out.println("3. 执行了请求处理的方法: execute");
		return SUCCESS;
	}
}
```

> 在配置文件中注册拦截器

```xml
<interceptor name="helloInterceptor" class="com.theliang.interceptor.HelloInterceptor"></interceptor>
```

> 为Action指定拦截器

```xml
<action name="helloInterceptor" class="com.theliang.interceptor.HelloAction">
			<result name="success">/index.jsp</result>
		</action>
```

> 这样做了以后，就会出现一个问题，struts2中为一个action指定拦截器后，默认的defaultStack中的拦截器就不起作用了，也就是说struts2的众多核心功能都使用不了了（struts2的许多核心功能都是通过拦截器实现的），为了解决这个问题，引入拦截器栈，先使用系统默认的拦截器，然后再来使用自定义的拦截器

```xml
<interceptors>
	<interceptor name="helloInterceptor" class="com.theliang.interceptor.HelloInterceptor"></interceptor>
	<interceptor-stack name="helloStack">
		<interceptor-ref name="defaultStack"></interceptor-ref>
		<interceptor-ref name="helloInterceptor"></interceptor-ref>
	</interceptor-stack>
</interceptors>
```

> 如果希望包下的所有action都使用自定义的拦截器，可以把拦截器设置为默认拦截器

```xml
<default-interceptor-ref name="helloStack"></default-interceptor-ref>
```

## 对struts.xml 的配置

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE struts PUBLIC
          "-//Apache Software Foundation//DTD Struts Configuration 2.0//EN"
          "http://struts.apache.org/dtds/struts-2.0.dtd">
<struts>
	<package name="helloInterceptor" extends="struts-default">

	<!-- 【拦截器配置】 -->
		<interceptors>
			<!-- 在配置文件中注册拦截器，配置用户自定义的拦截器 -->
			<interceptor name="helloInterceptor" class="com.theliang.interceptor.HelloInterceptor"></interceptor>
			
			<!-- 自定义一个栈：要引用默认栈、自定义的拦截器 -->
			<interceptor-stack name="helloStack">
				<!-- 引用一个默认栈（一定要放到第一行） -->
				<interceptor-ref name="defaultStack"></interceptor-ref>
				<!-- 引用自定义拦截器 -->
				<interceptor-ref name="helloInterceptor"></interceptor-ref>
			</interceptor-stack>
		</interceptors>
	
		<!-- 【执行拦截器】 -->
		<default-interceptor-ref name="helloStack"></default-interceptor-ref>
	
		<!-- Action配置 ，为Action指定拦截器-->
		<action name="helloInterceptor" class="com.theliang.interceptor.HelloAction">
			<result name="success">/index.jsp</result>
		</action>
	</package>
</struts>
```


