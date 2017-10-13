[TOC]

# Java国际化

Java程序的国际化主要通过如下3个类完成

> Java.util.ResourceBundle:用于加载资源包  
> Java.util.Locale:对应一个特定的国家/地区、语言环境  
> Java.text.MessageFormat:用于将消息格式化  

## 资源文件

为了实现程序的国际化，必须先提供程序所需要的资源文件。资源文件的内容是很多的 key-value对，其中key是程序使用的部分，而value是程序的显示部分。

### 资源文件的命名可以是如下3种形式

> baseName_language_country.properties  
> baseName_language.properties  
> baseName.properties  

其中baseName是资源文件的基本名称，用户可以自由定义，而language和country都不可随意变化，必须是Java所支持的语言和国家。

### Java支持的国家和语言

事实上，Java不可能支持所有的国家和语言，可以通过**Locale类的getAvailableLocale方法**获取支持的，该方法返回一个Locale数组，该数组中包含了所有支持的国家和语言。

```java
 Locale[] locales = Locale.getAvailableLocales();  
```

```java
package com.ascent.i18n.test;  
import java.util.*;  
public class LocalTest {  
    public static void main(String[] args) {  
        Locale [] locales = Locale.getAvailableLocales();  
          
        for(Locale locale:locales){  
        //输出所有支持的国家 
        System.out.print(locale.getDisplayCountry()+":"+locale.getCountry());  
              
        //输出所有支出的语言 
        System.out.println(locale.getDisplayLanguage()+":"+locale.getLanguage());  
        }  
    }  
}  
```

### 使用资源文件完成程序国际化

先编写两个资源文件，例如：

		resource_zh_CN.properties
		title=标题

		 resource_en_US.properties
         title=title

国际化实例

```java
package com.ascent.i18n.test;  
import java.util.*;  
public class ResourceBundleTest {  
    public static void main(String[] args) {  
        //设置本地区语言（默认）  
        Locale locale = Locale.getDefault();  
        //可以使用Local的常量设置具体的语言环境  
        //Locale locale = Locale.US;  
        //根据地区不同加载不同的资源文件  
        ResourceBundle rb = ResourceBundle.getBundle("resource", locale);  
        //根据key获得value值  
        String title = rb.getString("title");  
        System.out.println(title);  
    }  
}
```

以上在资源文件配置的消息都是简单的消息，假如消息中含有参数，即有参数占位，我们
如何实现呢？例如下面消息：

	英文：hello=Hello,{0}!Today is {1}.
	中文：hello=你好，{0}!今天是{1}.

此时，我们需要使用**MessageFormat类**，该类有个很有用的静态的方法
`format(String pattern,Object… values) `
返回后面多个参数值填充前面pattern字符串，其中pattern字符串就是一个带占位符的字符串。 

```java
package com.ascent.i18n.test;  
import java.util.Date;  
import java.util.Locale;  
import java.util.ResourceBundle;  
import java.text.*;  
public class MessageFormatTest {  
    public static void main(String[] args) {  
        Locale locale = Locale.getDefault();  
        ResourceBundle rb = ResourceBundle.getBundle("resource", locale);  
        String hello = rb.getString("hello");  
        String result = MessageFormat.format(hello, "焦学理",new Date());  
        System.out.println(result);  
    }  
}  
```

# Struts 2的国际化入门

Struts 2国际化是建立在Java国际化的基础之上，一样也是通过提供不同国家/语言环境的消息资源，然后通过**ResourceBundle**加载指定Locale对应的资源文件，再取得该资源文件中指定key对应的消息---整个过程与Java程序的国际化完全相同，只是Struts2框架对Java程序国际化进行了进一步封装，从而简化了应用程序的国际化。

## Struts 2中加载全局资源文件

Struts 2提供了很多加载国际化资源文件的方式，最简单.最常用的就是加载全局的国际化资源文件，加载全局的国际化资源文件的方式通过配置常量来实现。不管在**struts.xml**文件中配置常量，还是在**struts.properties**文件中配置常量，只需要配置**struts.custom.i18n.resources**常量即可。

配置struts.custom.i18n.resources常量时，该常量值为全局国际化资源文件的baseName。


假如系统需要加载的国际化资源文件的baseName为properties/message，则我们可以在**struts.properties**文件中指定如下一行：
        `struts.custom.i18n.resources= properties.message`


 或者更好的做法是在`struts.xml`文件中配置如下的一个常量：

```xml
<!-- 定义资源文件的位置和类型 -->
<constant name="struts.custom.i18n.resources" value="properties/message"/>
```

## 访问国际化资源

Struts2即可以在JSP页面中通过标签输出国际化消息，也可以在Action类中输出国际化消息，不管采用哪种方式，Struts2都提供了支持, 使用起来非常简单。

 Struts2访问国际化消息主要在如下3种方式：

1. 为了在JSP页面中输出国际化消息，可以使用Struts2的`<s:text…/>`标签，该标签可以指定一个name 属性,该属性指定了国际化资源文件中的key。

2. 为了在Action类中访问国际化消息,可以使用**ActionSupport类的getText方法**，该方法可以接受一个name 参数，该参数指定了国际化资源文件中的key。

3. 为了在该表单元 Label里输出国际化信息，可以为该表单标签指定一个key属性， 该key指定了国际化资源文件中的key。

以项目中的登陆国际化为例，下面我们要做的是在项目中src下添加properties文件夹，下面添加资源文件 

> message_zh_CN.properties  
> message_en_US.properties

    message_zh_CN.properties内容：
    	welcome＝欢迎  
		loginPage=登陆页面  
		errorPage=错误页面
		welcomePage=成功欢迎页面
		showBooksPage=图书展现页面

	message_en_US.properties内容：    
		welcome＝Welcome    
		loginPage=loginPage
		errorPage=errorPage
		welcomePage=welcomePage
		showBooksPage=showBooksPage

```html
<%@ page language="java" contentType="text/html; charset=utf-8"%>  
<%@ taglib uri="/struts-tags" prefix="s"%>  
<html>
    <head>  
        <!-- 使用s:text 标签输出国际化消息-->  
        <title><s:text name="loginPage"/></title>  
    </head>  
    <body>  
        <h3><s:text name="loginTip"/></h3>  

         <!-- 第一种获取方式 -->  
	    <s:text name="welcome">  
	        <s:param><s:property value="name"/></s:param>  
	        <s:param>游玩</s:param>  
	    </s:text><br>  

        <!-- 第二种获取方式 -->  
	    <s:textfield name="" value="" key="welcome"></s:textfield><br>

	     <!-- 第三种获取方式:在action中通过getText("welcome")获取数据，然后放到request域中，在jsp中通过el表达式读取 -->  
	    ${msg }<br>

        <!-- 在表单元素中使用key来指定国际化消息的key--> 
        <s:form action="Login" method="post">  
            <s:textfield name="username" key="user"/>  
            <s:password name="password" key="password"/>  
            <s:submit name="submit" key="submit" />  
        </s:form>
    </body>
</html> 
```

上面的JSP页面中使用了`<s:text…/>`标签来**直接输出国际化信息**,也**通过在表单元素中指定key属性**来输出国际化信息.通过这种方式,就可以完成JSP页面中普通文本.表单元素标签的国际化

如果为了在Action中访问国际化消息,则可以利用ActionSupport类的getText方法

```java
public class LoginAction extends ActionSupport{  
    public String execute(){  
        if(getUsername().equals("ascent")&& getPassword().equals("ascent")){  
        ActionContext.getContext().getSession().put("user", this.getUsername());  
        return SUCCESS;  
        }  
        return ERROR;  
    }  
    //完成输入校验需要重写的validate方法（读取资源文件getText(String str)）  
    public void validate(){

        //调用getText方法取出国际化信息  
        if(getUsername()==null||"".equals(this.getUsername().trim())){  
            this.addFieldError("username", this.getText("username.required"));  
        }  
        if(this.getPassword()==null||"".equals(this.getPassword().trim())){  
            this.addFieldError("password", this.getText("password.required"));  
        }  
    }  
}
```

通过在Action类中调用ActionSupport类的getText通过这种方式，就可以取得国际化资源文
件中的国际化消息。通过这种方式，即使Action需要设置在下一个页面显示的信息，也无需
直接设置字符串常量，而是使用国际化消息的key来输出，从而实现程序的国际化。

## 参数化国际化字符串(带有占位符)

许多情况下，我们都需要在运行时（runtime）为国际化字符插入一些参数，例如在输入验证
提示信息的时候。在Struts 2.0中，我们通过可以方便地做到这点。

1. 如果需要在JSP页面中填充国际化消息里的占位符，则可以通过在`<s:text…/>`标签中使
用多个`<s:param…/>`标签来填充消息中的占位符。第一个`<s:param…/>`标签指定第一个占位
符值，第二个`<s:param…/>`标签指定第二个占位符值……依此类推。

实例中国际化资源文件中有如下国际化消息：

	#带占位符的国际化信息
	welcomeTip=欢迎，{0},您已经登陆成功！ 

2. 如果需要在Action中填充国际化消息里的占位符，则可以通过在调用getText方法时使
用getText（String aTextName,List args）或getText(String key, String[] args)方法来填充
占位符。该方法的第二个参数既可以是一个字符串数组，也可以是字符串组成的List对象，从
而完成对占位符，字符串数组、字符串集合中第二个元素将填充第二个占位符，依此类推。

为了在Action类中输出占位符的消息，我们在Action类中调用ActionSupport类的getText方
法，调用该方法时，传入用于填充占位符的参数值。访问该带占位符消息的Action类如下:

```java
public String execute(){  
    if(getUsername().equals("ascent")&& getPassword().equals("ascent")){  
        //调用getText方法取出国际化信息，使用字符串数组传入占位符的参数值（request范围）  
        ActionContext.getContext().put("user", this.getText("welcomeTip",new String[]{this.getUsername()}));  
        return SUCCESS;  
    }  
    return ERROR;  
}
```

通过上面的带参数的getText方法，就可以为国际化消息的占位符传入参数了。

为了在JSP页面中输出带两个占位符的国际化消息，只需要为`<s:text…/>`标签指定`<s:param…/>`子标签即可。下面是welcome.jsp页面的代码： 

```html
<body>   
    <!--使用s:text标签输出welcomeTip对应的国际化信息-->  
    <s:text name="welcomeTip">  
            <!--使用s:param为国际化信息的占位符传入参数-->  
        <s:param><s:property value="username"/></s:param>  
     </s:text>  
    <br><br>  
     <!-- 输出request范围内的user值（资源文件取值）-->  
     request：${requestScope.user}  
    <br>  
        <a href="getBooks.action"><s:text name="welcomeLink"/></a>  
</body>
```

上面的页面使用`${requestScope.user}`输出的是Action类中取出的国际化消息，而通过
`<s:text…/>`标签取出的是另一个国际化消息，且使用了`<param…/>`标签为该国际化消息
的占位符指定了占位符值。