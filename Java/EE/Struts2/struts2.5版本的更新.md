
* 2.5.2版本不再提供xwork.jar ，整合到了 `struts-core` 包中。

* 方法不能访问的问题，需要在每个**action配置文件**中加上 `strict-method-invocation="false"`

```xml
<package name="login" namespace="/login" extends="struts-default" strict-method-invocation="false">
```

并修改配置文件头部为2.5版本的

```xml
<!DOCTYPE struts PUBLIC
"-//Apache Software Foundation//DTD Struts Configuration 2.5//EN"
"http://struts.apache.org/dtds/struts-2.5.dtd">
```

* session失效的问题，针对weblogic server，增加session-descriptor节点

```xml
<?xml version="1.0" encoding="UTF-8"?>
<weblogic-web-app xmlns="http://www.bea.com/ns/weblogic/90">

	<context-root>/ynwjnw</context-root>

	<container-descriptor>
		<servlet-reload-check-secs>-1</servlet-reload-check-secs>
		<prefer-web-inf-classes>true</prefer-web-inf-classes>
	</container-descriptor>

	<session-descriptor>
		<cookie-name>JSESSIONID1</cookie-name>
	</session-descriptor>

</weblogic-web-app>
```

* 2.5.2版本jdk要求1.7 5，**`web.xml`**中把

```xml
org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter
```

修改为：

```xml
org.apache.struts2.dispatcher.filter.StrutsPrepareAndExecuteFilter
```

