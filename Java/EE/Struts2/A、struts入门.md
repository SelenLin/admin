[TOC]

# struts2简介

struts2是在webwork2基础上发展而来的。和struts1一样，struts2也属于MVC框架。不过有一点需要注意的是：struts2和struts虽然名字很相似，但是在两者在代码编写风格上几乎是不一样的。那么既然有了struts1，为什么还要推出struts2。主要的原因是struts2有以下优点：

> 1. 在软件设计上struts2没有像struts1那样跟servlet API和struts API有着紧密的耦合，struts2的应用可以不依赖于servlet API和struts API。struts2的这种设计属于无侵入式设计，而struts1却属于侵入式设计。
> 
> 2. struts2提供了拦截器，利用拦截器可以进行AOP编程，实现如权限拦截等功能。
> 
> 3. struts2提供了类型转换器，可以把特殊的请求参数转化成需要的类型。在struts1中，如果我们要实现同样的功能，就必须向struts1的底层实现BeanUtil注册类型转换器才行。
> 
> 4. struts2提供支持多种表现层技术，如：jsp、freemarker、velocity等。
> 
> 5. struts2的输入校验可以对指定的方法进行校验，解决了struts1长久之痛。
> 
> 6. 提供了全局范围、包范围和Action范围的国际化资源文件实现。

---

> jre与jdk：
>> jre不包含调试   
>> jdk包含调试
>    
>    
> Struts:
>> Struts1  ：看可以不学   
>> Struts2  :  学  
> 二者无任何联系。
> 

> Struts1和Struts2的区别：
> 
> Struts1的action和具体的Struts的环境绑定，离不开Servlet环境
> 
> Struts2代替掉servlet，只需一个java类即可。   
> 
> Struts1里Action对象只有一个，任何人访问都只有一个,就需要做很好的线程同步的控制，Struts2每次访问一个Action时都会New一个对象出来，永远不会有线程同步的问题。   

# struts2开发环境搭建

搭建struts2 的开发环境的时，一般都会按如下的步骤：

## 1. 引入struts2需要的jar文件

	commons-fileupload-1.2.2.jar 	【文件上传相关包】
	commons-io-2.0.1.jar
	struts2-core-2.3.4.1.jar 	【struts2核心功能包】
	xwork-core-2.3.4.1.jar 		【Xwork核心包】
	ognl-3.0.5.jar 			【Ognl表达式功能支持表】
	commons-lang3-3.1.jar 		【struts对java.lang包的扩展】
	freemarker-2.3.19.jar 		【struts的标签模板库jar文件】
	javassist-3.11.0.GA.jar 	【struts对字节码的处理相关jar】

这些jar文件，可以从struts2自带的示例项目中拷贝，粘贴到`WebRoot/WEB-INF/lib`下面）

## 2. 配置web.xml

> 在`WEB-INF/web.xml`中加入struts2框架启动配置，具体的方法是在`WEB-INF/web.xml`中加入如下的代码：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd" id="WebApp_ID" version="3.0">
	<welcome-file-list>
		<welcome-file>index.jsp</welcome-file>
	</welcome-file-list>

	<!-- 引入struts核心过滤器 -->
  	<filter>
    	<filter-name>struts2</filter-name>
    	<filter-class>org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter</filter-class>
  	</filter>
  	<filter-mapping>
    	<filter-name>struts2</filter-name>
    	<url-pattern>/*</url-pattern>
  	</filter-mapping>
</web-app>
```

> 从上面可以看出，struts2框架是通过filter启动的。在`StrutsPrepareAndExecuteFilter`的init()方法中读取类路径下默认的配置文件struts.xml完成初始化操作。
> 
> 注意：struts2读取到struts.xml中的内容后，以javabean的形式保存在内存中，以后struts2对用户的每次请求处理将使用内存中的数据，而不是每次都读取struts.xml文件。

## 3. 开发Action

```java
// 开发action： 处理请求
public class HelloAction extends ActionSupport {
	
	// 处理请求
	public String execute() throws Exception {
		System.out.println("访问到了action，正在处理请求");
		System.out.println("调用service");
		return "success";
	}
}
```

> struts2中使用包来管理action。包的作用和java中类包是非常类似的，它主要用于管理一组业务功能相关的action。在实际应用中，应该把一组业务功能相关的action放在同一个包下面。
> 
> 配置包是必须指定name属性，该name可以随意取名，但是必须唯一，它不对应java的类包，如果其他类要继承该包，必须使用该属性进行使用。包的namespace属性用于定义包的命名空间，命名空间作为该action路径的一部分，如访问上面的例子中的action的路径为：/upload/upload.action。namespace属性可以不用配置，如果不指定该属性，默认的命名空间为“”（空字符串）。
> 
> 通常每个包都必须继承struts-default包，因为struts很多核心的功能都是在这个包中定义的拦截器实现的，如：从请求中把请求参数封装到action、文件上传和数据验证等功能搜是通过拦截器实现的。struts-default包中定义了这些拦截器和result类型。换句话说，当包继承了strtus-default包才能使用struts提供的核心功能。struts-default包是在struts2-core-2.x.x.x.jar文件中的struts-default.xml中定义的。struts-default.xml是struts2的默认配置文件，struts2每次都会自动加载struts-default.xml文件。
> 
> 包还可以通过abstract=“true”定义为抽象包，抽象包中不能包含action。
> 
> 注意，在配置文件struts.xml中没有提示的解决办法：window->preference->xml catalog中添加struts-2.0.dtd文件，key type为URI，key为http://struts.apache.org/dtds/struts-2.0.dtd。

### action名称的搜索顺序

1. 获得请求的URI，例如uri是:http://server/struts2/path1/path2/path3/test.action

2. 首先寻找'`namespace`'为'`/path1/path2/path3`'的package，如果不存在这个package，就转第三步，如果存在这个package，则在这个package中寻找名字为test的action，当在该package中找不到action时就到默认namespace的package中寻找（默认package的命名空间为空字符串“”），如果在默认的package中还找不到该action，页面提示找不到action。

3. 寻找'`namespace`'为'`/path1/path2`'的package，如果不存在这个package，则转第四步，如果存在这个package，则在这个package中寻找名字为test的action，当在该package中找不到action时就到默认namespace的package中寻找（默认package的命名空间为空字符串“”），如果在默认的package中还找不到该action，页面提示找不到action。

4. 寻找'`namespace`'为'`/path1`'的package，如果不存在这个package，则转第五步，如果存在这个package，则在这个package中寻找名字为test的action，当在该package中找不到action时就到默认namespace的package中寻找（默认package的命名空间为空字符串“”），如果在默认的package中还找不到该action，页面提示找不到action。

5. 寻找namespace为`/`的package，如果存在这个package，则在这个package中寻找名字为test的action，当在该package中找不到action或不存在这个package时就到默认namespace的package中寻找（默认package的命名空间为空字符串“”），如果在默认的package中还找不到该action，页面提示找不到action。

## 4. 配置struts.xml

编写struts2的配置文件（可以从struts2自带的示例项目中拷贝`struts.xml`，粘贴到src目录下，然后在这个基础上按照自己的需要来更改）

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE struts PUBLIC
          "-//Apache Software Foundation//DTD Struts Configuration 2.0//EN"
          "http://struts.apache.org/dtds/struts-2.0.dtd">
<struts>
    <package name="xxxx" namespace="/" extends="struts-default">
    	<action name="hello" class="com.theliang.action.HelloAction" method="execute">
    		<result name="success">/hello.jsp</result>
    	</action>
    </package> 
</struts>
<!--
1.如果没有为action指定class，默认的class是ActionSupport。

2.如果没有为action指定method，默认执行action中的execute方法。

3.如果没有为result指定name属性，默认值为success。
-->
```

> 其他的一些配置

```xml
<!--引入其他配置文件，可以把每个功能模块独立到一个xml配置文件中-->
    <include file="struts-User.xml"></include>

 <!--常量值-->
    <constant name="struts.devMode" value="true" /> 
    <constant name="struts.action.extension" value="action" />
<package name="default" namespace="/" extends="struts-default">

<!-- 注册拦截器 -->
    <interceptors>
        <interceptor name="mytimer" 
        class="com.peng.interceptor.TimerInterceptor"></interceptor>
    </interceptors>

<!-- 全局 result 配置 -->   
    <global-results>
        <result name="inplut">/error.jsp</result>
        <result name="error">/error.jsp</result>
    </global-results>

    <action name="timer" class="com.peng.action.TimerAction">
        <!-- 默认成功 -->
        <result>/success.jsp</result>
        <!-- 为Action显式引用拦截器后，默认的拦截器defaultStack不再生效，需要手工引用 -->
        <interceptor-ref name="defaultStack"></interceptor-ref>
        <!-- 引用拦截器 -->
        <interceptor-ref name="mytimer"></interceptor-ref>
        <!-- 参数设置，对应action中的get/set方法 -->
        <param name="url">http:www.qq.com</param>
    </action>
</package>
```

> 常量值

```xml
<!-- 请求数据编码，指定全局国际化资源文件mess。mess.properties在src目录下 -->
<constant name="struts.custom.i18n.resources" value="mess"/>

<!-- 指定默认编码集,作用于HttpServletRequest的setCharacterEncoding方法 和freemarker 、velocity的输出-->
<constant name="struts.i18n.encoding" value="UTF-8"/>

<!-- 修改struts默认的访问后缀，自定义后缀修改常量，*.do,*.action都会被识别。无后缀不能被识别-->
<constant name="struts.action.extension" value="do,go"/>
<!-- 如果想将无后缀也识别，请在上处常量设置value="do,go," --

<!-- 设置浏览器是否缓存静态内容,默认值为true(生产环境下使用),开发阶段最好关闭--> 
<constant name="struts.serve.static.browserCache" value="false"/>

<!-- 当struts的配置文件修改后,系统是否自动重新加载该文件,默认值为false(生产环境下使用),开发阶段最好打开 -->
<constant name="struts.configuration.xml.reload" value="true"/>

<!-- 开发模式下使用,这样可以打印出更详细的错误信息 -->
<constant name="struts.devMode" value="true" />

<!-- 默认的视图主题 -->
<constant name="struts.ui.theme" value="simple" />

<!-- 与spring集成时，指定由spring负责action对象的创建 -->
<constant name="struts.objectFactory" value="spring" />

<!--  该属性设置Struts 2是否支持动态方法调用，该属性的默认值是true。如果需要关闭动态方法调用，则可设置该属性
为 false-->
<constant name="struts.enable.DynamicMethodInvocation" value="false"/>

<!-- 上传文件的40M大小限制-->
<constant name="struts.multipart.maxSize" value="41943040"/>
```

# Struts2执行流程

> 服务器启动
> 
		1. 加载项目web.xml
		2. 创建Struts核心过滤器对象， 执行filter ->  init()

			struts-default.xml,     核心功能的初始化
			struts-plugin.xml,      struts相关插件
			struts.xml              用户编写的配置文件

> 访问
> 
		3. 用户访问Action, 服务器根据访问路径名称，找对应的aciton配置, 创建action对象
		4. 执行默认拦截器栈中定义的18个拦截器
		5. 执行action的业务处理方法

## struts-default.xml 详解

目录：`struts2-core-2.3.4.1.jar/ struts-default.xml`   

内容：   

	1. bean节点指定struts在运行的时候创建的对象类型   
	2. 指定struts-default包  【用户写的package(struts.xml)一样要继承此包 】   
		package  struts-default  包中定义了：  
			a.  跳转的结果类型   
				dispatcher    	转发，不指定默认为转发  
				redirect       	重定向  
				redirectAction  重定向到action资源  
				stream        	(文件下载的时候用) 

			b. 定义了所有的拦截器  
				  定义了32个拦截器！  
				  为了拦截器引用方便，可以通过定义栈的方式引用拦截器，此时如果引用了栈，栈中的拦截器都会被引用!
				
				defaultStack
					默认的栈，其中定义默认要执行的18个拦截器！  

			c. 默认执行的拦截器栈、默认执行的action
				<default-interceptor-ref name="defaultStack"/>
				<default-class-ref class="com.opensymphony.xwork2.ActionSupport" />

	<interceptor name="prepare" class="com.opensymphony.xwork2.interceptor.PrepareInterceptor"/>
	<interceptor name="params" class="com.opensymphony.xwork2.interceptor.ParametersInterceptor"/>

