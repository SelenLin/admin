[TOC]

# OGNL概述

       OGNL是Object Graphic Navigation
       Language（对象图导航语言）的缩写，它是一个开源项目。
       Struts2框架使用OGNL作为默认的表达式语言。

# OGNL优势

1. 支持对象方法调用，如`xxx.doSomeSpecial()`； 
2. 支持类静态的方法调用和值访问，表达式的格式:
         `@[类全名（包括包路径）]@[方法名 | 值名]`，例如：
         `@java.lang.String@format('foo %s', 'bar')`
         或`@tutorial.MyConstant@APP_NAME`； 
3. 支持赋值操作和表达式串联，如price=100, discount=0.8,
         calculatePrice()，这个表达式会返回80； 
4. 访问OGNL上下文（OGNL context）和ActionContext； 
5. 操作集合对象。

# 和EL表达式的区别

> 它提供了一些平时需要的功能:
> 
（1）支持对象方法调用，如`xxx.save()`   
（2）支持类静态方法调用和值访问，表达式的格式为@全类名（包括包路径）@方法名(参数)，如：`@java.lang.String@format('foo%s','bar')`或`@java.util.Math@PI`  
（3）操作集合对象   

# 使用Ognl

* 当struts2接受一个请求时，会迅速创建出ActionContext，ValueStack，action，然后把action存放进ValueStack，所以action的实例变量可以被ognl访问。

* 访问上下文中的对象需要使用`#`标注命名空间，如`#application`，`#session`等。

* ognl有一个根对象（**root对象**），在struts2中根对象就是ValueStack，如果要访问**根对象**中对象的属性，则**可以省略#**命名空间，直接访问该对象的属性即可。

* 在struts2中，根对象ValueStack的实现为`OgnlValueStack`,该对象不是只存放单个值，而是存放一组对象。在OgnlValueStack中有一个List类型的root变量，就是使用它存放一组对象。

* 在root变量中处于第一位的对象叫做栈顶对象。通常在OGNL表达式里面直接写上属性的名称即可访问root变量里对象的属性，搜索顺序是从栈顶对象开始，依次往下搜索，直到找到为止。

* 需要注意的一点是，struts2中ognl表达式要配合**struts标签**才可以使用。如：`<s:property value="name"/>`

* 由于ValueStack是Struts2中的ognl的根对象，如果用户需要访问ValueStack中的对象，在jsp页面可以通过下面的EL表达式访问ValueStack中对象的属性。

* ${foo}//获得ValueStack中某个对象的foo属性

<font color="blue">如果访问Context中的别的对象，由于它们不是根对象，所以在访问是需要添加#前缀</font>  
（1）**application对象**：用于访问ServletContext，例如*#application.userName*或*#application['userName']*，相当于调用session的getAttribute("userName")  
（2）**session对象**：用于访问HttpSession，例如*#session.userName*或*#session['userName']*，相当于调用ServletContext的getAttribute("userName")  
（3）**request对象**：用于访问HttpServletRequest对象，例如*#request.userName*或*#request['userName']*，相当于调用request的getAttribute("userName")  
（4）**parameters对象**：用于访问Http的请求参数，例如*#parameters.userName*或*#parameters['userName']*，相当于调用request的getParameter("userName")  
（5） **attr对象**：用于按 **page->request->session->application** 的顺序访问其属性。  

## OgnlContext

### 放入数据

```java
public void testOgnl()throws Exception{
	//创建一个Ognl上下文对象
	OgnlContext context = new OgnlContext();
	//放入数据
	
	context.put("cn","China");
	String value=(String)context.get("cn");
	System.out.println(value);
}
```

### 非根元素取值

> Ognl表达式语言取值，如果取非根元素表达式时必须使用“#”

```java
public void testOgnl2()throws Exception{
	//创建一个Ognl上下文对象
	OgnlContext context = new OgnlContext();
	//放入数据
	User user=new User();
	user.setId(132);
	user.setName("liang");
	
	//【往非根元素放入数据，取值的时候表达式要用“#”】
	context.put("user",user);
	context.put("id",12);
	
	//先构建一个ognl表达式，再解析表达式
	Object ognl=Ognl.parseExpression("#user.name");
	Object ognl1=context.get("user");
	
	//获取数据
	Object value=Ognl.getValue(ognl, context,context.getRoot());
	
	System.out.println(value);
	System.out.println(((User)ognl1).getId());
}
```

### 根元素取值

> Ognl表达式语言取值，取根元素的值，不用带“#”符号

```java
public void testOgnl3()throws Exception{
	//创建一个Ognl上下文对象
	OgnlContext context = new OgnlContext();
	//放入数据
	User user=new User();
	user.setId(132);
	user.setName("liang");
	
	//【往根元素放入数据，取值的时候表达式不要用“#”】
	context.setRoot(user);
	
	//先构建一个ognl表达式，再解析表达式
	Object ognl_id=Ognl.parseExpression("id");
	
	//获取数据
	Object value_id=Ognl.getValue(ognl_id, context,context.getRoot());
	
	System.out.println(value_id);
}
```

### 静态方法调用

```java
public void testOgnl4()throws Exception{
	//创建一个Ognl上下文对象
	OgnlContext context = new OgnlContext();

	//Ognl表达式语言，调用类的静态方法

	Object ognl=Ognl.parseExpression("@Math@floor(10.9)");
	// 由于Math类在开发中比较常用，所以也可以这样写
	Object ognl=Ognl.parseExpression("@@floor(10.9)");

	Object valueObject=Ognl.getValue(ognl,context,context.getRoot());
	
	System.out.println(valueObject);
}
```

## ValueStack

ValueStack实际是一个接口,在Struts2中利用OGNL时，实际上使用的是实现了该接口的OgnlValueStack类,这个类是Struts2利用OGNL的基础   

---

> 浅析值栈

ValueStack对象相当于一个栈，它贯穿整个Action的生命周期，每个Action类的对象实例都会拥有一个ValueStack对象
当Struts2接收到一个 ***.action** 请求后，并不是直接调用Action方法，而是先将Action类的相应属性放到ValueStack对象的顶层节点
值栈也位于内存中，它也是和`parameters`、`request`、`session`、`application`、`attr`对象放在一起的
值栈属于ONGL Context里面的根对象。也就是说它位于整个内存中最最重要的地方，所以叫根对象
根对象和另外五个对象是有区别的，根对象可以省写#号，比如`<s:property value="user.username"/>`
值栈的生命周期与request请求相关，每次请求产生一个值栈。默认所有的Action会被自动放到值栈里

---

> 服务器跳转时共用值栈

假设从一个Action11通过服务器跳转到Action22的话，就意味着这两个Action是共享一个值栈的，因为一次请求只使用一个值栈
这时内存中情况是这样的：首先接收到Action11请求后，会产生一个值栈，在栈顶存放Action11对象以及它所有的属性
然后经过服务器跳转到Action22，这时就会把Action22对象压入值栈的栈顶位置，此时Action11对象以及它的所有属性就位于栈底了

---

> 取值过程

栈的特征是后进先出。于是首先到栈顶的对象里查找是否存在这个属性，如果栈顶的Action22对象中不存在这个属性的话
它就会继续向下寻找直至栈底对象，一直查找是否存在这个属性
如果最后找到该属性的话，那么就会在JSP页面中通过`<s:property value="username"/>`输出属性值
如果在Action22和Action11都有一个同名的同类型的username属性的话，那么将输出Action22中的属性值
因为它是先从栈顶开始寻找属性的，值栈的特征就是后进先出，但有个前提：请求过程是通过服务器跳转的

### 获取值栈对象的两种方法

```java
private void getVs() {
	//获取值栈对象，方式一：
	//通过request对象中的请求头获取
	HttpServletRequest request = ServletActionContext.getRequest();
	ValueStack vs1=(ValueStack)request.getAttribute("struts.valueStack");

	//获取值栈对象，方式二：
	//通过ActionContext对象直接获取
	ActionContext ac=ActionContext.getContext();
	ValueStack vs2=ac.getValueStack();

	System.out.println(vs1==vs2);//true，说明这两种方法获取的对象是相同的
}
```
