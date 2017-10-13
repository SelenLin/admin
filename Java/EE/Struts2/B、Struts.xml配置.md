[TOC]

# struts.xml 配置详解

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE struts PUBLIC
          "-//Apache Software Foundation//DTD Struts Configuration 2.0//EN"
          "http://struts.apache.org/dtds/struts-2.0.dtd">
<struts>
    <package name="xxxx" namespace="/" extends="struts-default">
    	<!-- 
     	name	只配置了访问路径名称
     	class	默认执行的action在struts-default有配置，直接返回的是return SUCCESS;
     			<default-class-ref class="com.opensymphony.xwork2.ActionSupport"/>
     	method	默认为execute
     			默认方法execute返回值为SUCCESS，对应的页面会去全局视图查找
      -->
    	<action name="hello" class="com.theliang.action.HelloAction" method="execute">
    		<result name="success">/hello.jsp</result>

    		<!-- 返回结果标记success对应的页面，如果在当前action中没有配置 
     			所以会去全局跳转视图中查找
     		 -->

    	</action>
    </package> 
</struts>
<!--
1.如果没有为action指定class，默认的class是ActionSupport。

2.如果没有为action指定method，默认执行action中的execute方法。

3.如果没有为result指定name属性，默认值为success。
-->
```

## 其他的一些配置

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

## 常量值

```xml
<!-- 请求数据编码，指定全局国际化资源文件mess。mess.properties在src目录下 -->
<constant name="struts.custom.i18n.resources" value="mess"/>

<!-- 指定默认编码集,作用于HttpServletRequest的setCharacterEncoding方法 和freemarker 、velocity的输出-->
<constant name="struts.i18n.encoding" value="UTF-8"/>

<!-- 修改struts默认的访问后缀，自定义后缀修改常量，*.do,*.action都会被识别。无后缀不能被识别-->
<constant name="struts.action.extension" value="do,go"/>
<!-- 如果想将无后缀也识别，请在上处常量设置value="do,go," -->

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

