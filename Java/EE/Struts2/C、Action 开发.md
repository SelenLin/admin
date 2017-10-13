[TOC]

# Action的搜索顺序

如果这个路径可以

> http://localhost:8080/struts2/helloworld.action

则

> http://localhost:8080/struts2/aaa/bbb/helloworld.action

也可以

---

1. 获得请求的URI，例如uri是:http://server/struts2/path1/path2/path3/test.action

2. 首先寻找'`namespace`'为'`/path1/path2/path3`'的package，如果不存在这个package，就转第三步，如果存在这个package，则在这个package中寻找名字为test的action，当在该package中找不到action时就到默认namespace的package中寻找（默认package的命名空间为空字符串“”），如果在默认的package中还找不到该action，页面提示找不到action。

3. 寻找'`namespace`'为'`/path1/path2`'的package，如果不存在这个package，则转第四步，如果存在这个package，则在这个package中寻找名字为test的action，当在该package中找不到action时就到默认namespace的package中寻找（默认package的命名空间为空字符串“”），如果在默认的package中还找不到该action，页面提示找不到action。

4. 寻找'`namespace`'为'`/path1`'的package，如果不存在这个package，则转第五步，如果存在这个package，则在这个package中寻找名字为test的action，当在该package中找不到action时就到默认namespace的package中寻找（默认package的命名空间为空字符串“”），如果在默认的package中还找不到该action，页面提示找不到action。

5. 寻找namespace为`/`的package，如果存在这个package，则在这个package中寻找名字为test的action，当在该package中找不到action或不存在这个package时就到默认namespace的package中寻找（默认package的命名空间为空字符串“”），如果在默认的package中还找不到该action，页面提示找不到action。

# 动态方法调用

动态方法调用就是为了解决一个Action对应多个请求的处理时Action太多

> 一个Action就一个execute()方法，一个模块写一个action，action只有一个默认的execute()方法

## 三种方式解决

### 1.  指定method属性

建立一个action包，存放action，src下存放struts.xml.在WebContent下建立相应的jsp文件

> action中加入了两个方法 add ，update，都返回SUCCESS

    HelloWorldAction.java

```java
public class HelloWorldAction extends ActionSupport {
    public String add(){return SUCCESS;}
    public String update(){return SUCCESS;}
    @Override
    public String execute() throws Exception {
        System.out.println("执行Actoin");
        return SUCCESS;
    }
}
```

    struts.xml文件中做相应的更改

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE struts PUBLIC
          "-//Apache Software Foundation//DTD Struts Configuration 2.0//EN"
          "http://struts.apache.org/dtds/struts-2.0.dtd">
<struts>
     <package name="execute" extends="struts-default" namespace="/">

        <action name="helloworld"  class="com.struts2.action.HelloWorldAction">
            <result >/result.jsp</result>       
        </action>

        <action name="addAction" method="add" class="com.struts2.action.HelloWorldAction">
            <result >/add.jsp</result>
        </action>

        <action name="updateAction" method="update" class="com.struts2.action.HelloWorldAction">
            <result >/update.jsp</result>
        </action>   
    </package> 
</struts>
```

> http://localhost:8080/Struts2_Test/addAction 访问的是add.jsp   
> http://localhost:8080/Struts2_Test/updateAction 访问的是update.jsp

### 2.  感叹号方式

> (官方不推荐使用)

首先在`struts.xml`中开启一个常量

```xml
 <!-- 动态调用方法 -->
 <constant name="struts.enable.DynamicMethodInvocation" value="true"></constant>
```

更改struct.xml文件的action

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE struts PUBLIC
          "-//Apache Software Foundation//DTD Struts Configuration 2.0//EN"
          "http://struts.apache.org/dtds/struts-2.0.dtd">
<struts>
     <package name="execute" extends="struts-default" namespace="/">

        <action name="helloworld" method="{1}" class="com.struts2.action.HelloWorldAction">
            <result >/result.jsp</result>
            <result name="add">/add.jsp</result>
            <result name="update">/update.jsp</result>
        </action>
    </package> 
</struts>
```

更改HelloWorldAction ，**注意方法的return值要相应改变**

```java
public class HelloWorldAction extends ActionSupport {
    public String add(){
        return "add";
    }
    public String update(){
        return "update";
    }
    @Override
    public String execute() throws Exception {
        System.out.println("执行Actoin");
        return SUCCESS;
    }
}
```

> http://localhost:8080/Struts2_Test/helloworld.action 访问的是result.jsp   
> http://localhost:8080/Struts2_Test/helloworld!add.action 访问的是add.jsp   
> http://localhost:8080/Struts2_Test/helloworld!update.action 访问的是update.jsp   

### 3.  通配符方式

关闭感叹号的方式，设为 false

```xml
 <!-- 动态调用方法 -->
 <constant name="struts.enable.DynamicMethodInvocation" value="false"></constant>
```

更改struct.xml文件的action

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE struts PUBLIC
          "-//Apache Software Foundation//DTD Struts Configuration 2.0//EN"
          "http://struts.apache.org/dtds/struts-2.0.dtd">
<struts>
     <package name="execute" extends="struts-default" namespace="/">

        <action name="helloworld_*" method="{1}" class="com.struts2.action.HelloWorldAction">
            <result >/result.jsp</result>
            <result name="add">/{1}.jsp</result>
            <result name="update">/{1}.jsp</result>
        </action>
    </package> 
</struts>
```

更改HelloWorldAction

```java
public class HelloWorldAction extends ActionSupport {
    public String add(){
        return "add";
    }
    public String update(){
        return "update";
    }
    @Override
    public String execute() throws Exception {
        System.out.println("执行Actoin");
        return SUCCESS;
    }
}
```

> http://localhost:8080/Struts2_Test/helloworld.action 访问的是result.jsp   
> http://localhost:8080/Struts2_Test/helloworld_add.action 访问的是add.jsp

当然action也可匹配多个

```xml
<action name="*_*" method="{2}" class="com.struts2.action.{1}Action">
    <result >/result.jsp</result>
    <result name="add">/{2}.jsp</result>
    <result name="update">/{2}.jsp</result>
</action>
```

> http://localhost:8080/Struts2_Test/HelloWorld_add.action 访问的是add.jsp
> 
>大写的HelloWorld，因为是类的路径啊

