[TOC]

# Ognl标签

## $号的使用

1. 在国际化资源文件中，引用OGNL表达式
2. 在struts.xml文件中，引用OGNL表达式

* 举例：ognl.xml配置文件

```xml
<action name="ognlAction_*" class="com.theliang.ognl.OgnlAction" method="{1}">
      <result nme="success">/ognl/ognl.jsp?username=${#request.username}</result>
</action>
```

* 在ognl.jsp中获取携带的参数：

```html
<s:property value="%{#parameters.username[0]}"/>
```

## %号的使用

> “%”符号的作用是在当Struts2标签的属性值为OGNL表达式时OGNL表达式却被理解为字符串类型原样输出时，用于通知执行环境%{}里的是OGNL表达式

*  使用%{}可以取出保存在值堆栈中的Action对象，直接调用它的方法

> 举例

Struts2中的textfield标签主要用于在页面中显示一个文本输入框数据。类似input
`<s:textfield value="#request.username"></s:textfield>`
此时`#request.username`被理解为一个普通的字符串，因此直接显示。因为这里脱离了
运行OGNL的环境即：`<s:property value="OGNL表达式"/>`环境。
通知运行环境将`#request.username`以OGNL表达式运行：
`<s:textfield value="%{#request.username}"></s:textfield>`

所以为了方便使用%{}我们可以在任何地方都直接添加%{}来确保运行OGNL表达式：
       `<s:property value="%{#request.username}"/>`


* 如果Action继承了ActionSupport，那么在页面标签中可以使用`%{getText('key')}`获取国际化信息

## #号的使用

> OGNL中的#号可以取出堆栈上下文中存放的对象 

| 名称        | 作用                                              | 例子                                                             |
| :--         | :--                                               | :--                                                              |
| attr        | 用于按request>>session>>application顺序访问其属性 | #attr.userName相当于按顺序从三个范围读取userName属性直到找到为止 |
| request     | 包含当前HttpServletRequest的属性的Map             | #request.userName相当于request.getAttribute("userName")          |
| session     | 包含当前HttpSession的属性的Map                    | #request.userName相当于request.getAttribute("userName")          |
| application | 包含当前应用的ServletContext的属性的Map           | #request.userName相当于request.getAttribute("userName")          |
| parameters  | 包含当前HTTP请求参数的Map                         | #parameters.id[0]相当于request.getParameter("id")                |

> \#号还有一个作用就是在JSP页面中构建Map集合

格式：#{key:value,key:value...}

* 举例

```html
<s:radio list="#{'male':'男','female':'女'}" name="gender"></s:radio>

<s:iterator var="str" value="{'a','b'}">
	<s:property value="#str"/>
</s:iterator>

<s:iterator var="en" value="#{'cn':'china','usa':'American'}">
	<s:property value="#en.key"/>
	<s:property value="#en.value" />
</s:iterator>
```

## property标签

```html
<s:property value=“#name" default="a default value" />
	* default：可选属性， 如果需要输出的属性值为null，则显示该属性指定的值
	*  escape：可选属性，指定是否格式化HTML代码。
	*  value：   可选属性，指定需要输出的属性值，如果没有指定该属性，则默认输出ValueStack栈顶的值。
```

### 获取Action中的属性值或者Action中的对象的某某属性值

利用`<s:property/>`标签可以直接获取Action中的引用类型user里面的username属性
同样可以通过`user.address.addr`获取user中引用类型address中的addr属性的值
像这种一层一层往下传递的访问方式，即**所谓的导航**，也就是一步步的往下调用


### 调用Action的对象里面的普通方法

默认的会把Action放到值栈里面，而值栈在访问的时候，并不需要值栈的名字
当我们调用`<s:property value="user.getVOMethod()"/>`的时候
它会自动到值栈里面查找Action对象里面有没有user对象，然后它就发现有user
然后它就再找user里面有没有getVOMethod()方法，然后它发现有，于是调用getVOMethod()
实际上调用User中的getVOMethod()方法的过程与获取表单中的姓名密码的方式都是相同的
**都是到值栈里面查找**，找是否存在user对象，如果存在，接着查找user中是否存在某某属性或方法

### 调用Action中的静态方法

同样我们也可以在JSP页面中写一个OGNL表达式调用Action中的静态方法
调用Action中的静态方法时，与调用user对象的getVOMethod()方法的过程，是截然不同的
此时value的写法是固定的，**以@开头，后面跟上具体的包名，然后@加上静态方法**

比如`<s:property value="@com.theliang.action.LoginAction@getStatic()"/>`
另外user对象是LoginAction中的一个属性，这个属性会自动的放到值栈里面
**而值栈调用的时候，不用加上@或者包名等等**，所以直接user.getVOMethod()就可以了

### 调用JDK类中的静态方法

可以使用`<s:property value="@@floor(46.58)"/>`，或者"**@Math@floor(46.58)**"，输出floor()的执行结果
这就意味着如果不在@@中指定类的话，默认的就表示**java.lang.Math类**。
当前大多数情况下，我们都不会省略这个类，都会写全了的，然后在后面加上静态方法

### 集合的伪属性

OGNL能够引用集合的一些特殊的属性，这些属性并不是JavaBean模式，例如*size()*、*length()*  
当表达式引用这些属性时，OGNL会调用相应的方法，这就是伪属性
比如获取List的大小：`<s:property value="testList.size"/>`

| 对象        | 属性                                        |
| :--         | :--                                         |
| List        | size、isEmpty、iterator                     |
| Set         | size、isEmpty、iterator                     |
| Map         | size、isEmpty、keys、values                 |
| Iterator    | next、hasNext                               |
| Enumeration | next、hasNext、nextElement、hasMoreElements |

### 获取集合中元素的实质就是调用它的toString()方法

它还可以直接获取集合中的元素，事实上是在调用集合的toString()方法
所以我们可以根据实际情况通过重写集合的toString()方法来实现个性化输出
甚至它还可以像访问数组那样，直接**testList[2]**获取集合中的元素
但这种方法只适用于List，不适用于Map。因为Map的索引是key，不是数值
另外，由于HashSet中的元素是没有顺序的，所以也不能用下标获取单个元素

### 其它

1. 当OGNL取不到值的时候，它不会报错，而是什么都不显示
2. `<s:property value="[0]"/>`返回的是ValueStack中从上至下的所有的Object
    `<s:property value="[1]"/>`返回的是ValueStack中从上至下的第二个Object
3. `<s:property value="[0].username"/>`返回的是成员变量username的值
    假设ValueStack中存在两个Action的话，如果第一个Action如果没有username变量
    那么它会继续找第二个Action。那么在什么情况下ValueStack中会存在两个Action呢
    答案是在struts.xml中配置的是从一个Action通过`<result type="chain">`跳转到另一个Action
4. `<constant name="struts.ognl.allowStaticMethodAccess" value="true"/>`
    在Struts2.1.6中必须设置`struts.ognl.allowStaticMethodAccess`为true之后
    才允许使用OGNL访问静态方法。而在Struts2.0.11则无需设置，即可直接访问

### 测试

#### 实体类

> User.java

```java
package com.theliang.i_ognl;

public class User {

	private int id;
	private String name;
	private Address address=new Address("北京省","北京市");
	public User() {
		super();
	}
	public User(int id, String name) {
		super();
		this.id = id;
		this.name = name;
	}
	@Override
	public String toString() {
		return "User [id=" + id + ", name=" + name + "]";
	}

	// set get ...
}
```

> Address.java

```java
package com.theliang.ognl;

public class Address {

	private String province;
	private String city;
	public Address() {
		super();
	}
	public Address(String province, String city) {
		super();
		this.province = province;
		this.city = city;
	}

	//set get ...
}
```

#### StrutsTags.java

```java
package com.theliang.ognl;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import com.opensymphony.xwork2.ActionContext;
import com.opensymphony.xwork2.ActionSupport;
/**
 * struts的数据流转
 * @author Administrator
 *
 */
public class StrutsTags extends ActionSupport{

	@Override
	public String execute() throws Exception {
		//测试迭代标签
		
		List<User> list=new ArrayList<User>();
		
		Map<Integer,User> map=new HashMap<Integer, User>();
		
		//初始化
		for (int i = 1; i < 11; i++) {
			User user=new User(i,"ABC"+i);
			
			list.add(user);
			map.put(user.getId(),user);
		}
		
		//保存
		ActionContext.getContext().getContextMap().put("list",list);
		ActionContext.getContext().getContextMap().put("map",map);
		
		return super.execute();
	}
}
```

#### JSP-获取域对象值

```html
<%@ page language="java" import="java.util.*" pageEncoding="utf-8"%>
<%@taglib uri="/struts-tags" prefix="s" %>
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
  <head>
    <title>首页</title>
  </head>
  <body>
  		这是首页<br/>
  		${requestScope.request_data }<br/>
  		${sessionScope.session_data }<br/>
  		${applicationScope.application_data }${applicationScope.context_data }<br/>
  	
  	<br/><br/>ValueStack数据获取<br/>
  		<h4> 1.取根元素的值</h4> 
  	1.<s:property value="user.id"/><br/>
  	2.<s:property value="user.name"/><br/>
  	3.<s:property value="id"/><br/>
  	4.<s:property value="name"/><br/>
  	5.<s:property value="address.province"/><br/>
  	6.<s:property value="user.address.city"/><br/><br/>
  	
  		 <h4> 2.取非根元素的值</h4> 
  	1.<s:property value="#request.cn"/><br/>
  	2.<s:property value="#request.request_data"/><br/>
  	3.<s:property value="#session.session_data"/><br/>
  	4.<s:property value="#application.application_data"/><br/>
  	5.<s:property value="#application.context_data"/><br/><br/>
  	
  		<h4> 3.取attr的值(自动找request/session/application，找到后返回)</h4> 
  	1.<s:property value="#attr.cn"/><br/>
  	1.1.<s:property value="#cn"/><br/>
  	2.<s:property value="#attr.request_data"/><br/>
  	3.<s:property value="#attr.session_data"/><br/>
  	4.<s:property value="#attr.application_data"/><br/>
  	5.<s:property value="#attr.context_data"/><br/><br/>
  	
  		<h4> 4.取paramter请求参数的数据</h4>
  	<a href="${pageContext.request.contextPath}/vs?userName=liang&pwd=123645">测试</a><br/> 
  	<a href="${pageContext.request.contextPath}/index.jsp?userName=liang&pwd=123645">测试</a><br/> 
  	1.<s:property value="#parameters.userName"/><br/>	
  	2.<s:property value="#parameters.pwd"/><br/>	

  	<!-- struts的调试标签：可以查看值栈数据 -->
  	<s:debug></s:debug>
  </body>
</html>
```

#### JSP-获取集合数据

```html
<%@ page language="java" import="java.util.*" pageEncoding="utf-8"%>
<%@taglib uri="/struts-tags"  prefix="s"%>
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
  <head>
    <title>登录</title>
  </head>
  <style type="text/css">
  	.odd{background-color:red;}
  	.even{background-color:blue;}
  </style>
  <body>
  	<h3>一、list迭代</h3>
  	<table border="1">
  		<tr>
  			<td>编号</td>
  			<td>姓名</td>
  		</tr>
  		
  		<s:iterator var="user" value="#request.list" status="st">
	  		<tr  class=<s:property value="#st.even?'even':'odd'"/>>
	  			<td><s:property value="#user.id"/></td>
	  			<td><s:property value="#user.name"/></td>
	  		</tr>
  		</s:iterator>
  	</table><hr>

  	<h3>二、map迭代</h3>
  	<table border="1">
  		<tr>
  			<td>编号</td>
  			<td>姓名</td>
  		</tr>
  		
  		<s:iterator var="en" value="#request.map" status="st">
	  		<tr>
	  			<td><s:property value="#en.key"/></td>
	  			<td><s:property value="#en.value.name"/></td>
	  		</tr>
  		</s:iterator>
  	</table>
  	
  	<h3>Ognl表达式可以取值 ，也可以动态构建list&map集合</h3>
  	<s:iterator var="str" value="{'a','b'}">
  		<s:property value="#str"/>
  	</s:iterator>
  	
  	<s:iterator var="en" value="#{'cn':'china','usa':'American'}">
  		<s:property value="#en.key"/>
  		<s:property value="#en.value" />
  	</s:iterator>
  </body>
</html>
```

## set标签

set标签用于将某个值放入指定范围。
 
| 名称  | 必填  | 缺省值 | 类型          | 描述                                                                                                |
| :--   | :--   | :--    | :--           | :--                                                                                                 |
| name  | true  |        | String        | 变量名字。name,id和var表达的含义是一样的。name,id被var替代                                          |
| scope | false |        | String        | 变量作用域，可以为application,session,request,page,action。如果没有设置该属性，则默认放置在action中 |
| value | false |        | Object/String | 将会赋给变量的值。如果没有设置该属性,则将ValueStack栈顶的值赋给变量。                               |
| id    | false |        | Object/String | 用来标识元素的id。在ui和表单中为HTML的id属性                                                        |

举例

```html
<s:set value="#request.username"  var="xxx" scope="request" /><br>
<s:property value=“#request.xxx" /><br>

<s:set value="#request.username"  var="xxx" scope="page" /><br>
<s:property value="#attr.xxx" /><br>

<s:set value="#request.username"  var="xxx" /><br>
<s:property value="xxx" /><br>
```

## push标签

> push:将对象放入栈顶,不能放入其他范围,当标签结束,会从栈顶删除。  
> value：要push到堆栈中的值 。  

举例

```html
<s:push value="#request.username">
	<s:property/>
</s:push><br>
测试删除: <s:property/>
```

## bean标签(了解)

> bean标签: 实例化一个符合JavaBeans规范的class,标签体内可以包含几个Param元素,用于调用setter方法给此class的属性赋值  
> name:要被实例化的class名字(必须符合JavaBeans规范)  
> var： 赋给变量的值.放置在request作用域中，如果没有设置该属性,则对象被放置到栈顶

举例

```html
<s:bean  name="com.theliang.bean.Person" var="myperson">
   <s:param name="name" value="%{'zhang'}"></s:param>
   <s:param name="age" value="34"></s:param>
</s:bean><br>

<s:property value="#myperson.name"/>
```

## action标签

> Action:通过指定命名空间和action名称,该标签允许在jsp页面直接调用Action  
> 
> 		name:action名字(不包括后缀,如.action)  
> 		namespace:action所在命名空间  
> 		executeResult:Action的result是否需要被执行,默认值是false不执行  

举例

```xml
<package name="ognl"   namespace="/ognl" extends="struts-default" >
    <action name="ognlAction_*" class="com.theliang.ognl.OgnlAction" method="{1}">
       <result name="ognl">/ognl/ongl.jsp?msg=${msgxx}</result>
    </action>   
</package>   
```

```html
<s:action name="ognlAction_test" namespace="/ognl" executeResult="true" />
```

## sort标签

用来对一个可遍历的对象里的元素进行排序

| 名称       | 必填  | 缺省值 | 类型                 | 描述                                     |
| :--        | :--   | :--    | :--                  | :--                                      |
| comparator | false |        | java.util.Comparator | 在排序过程中使用的比较器                 |
| source     | false |        | String               | 将对之进行排序的可遍历对象               |
| var        | false |        | String               | 用来应用因排序而新生成的可遍历对象的变量 |

```html
s:sort<br>
<s:sort comparator="peopleComparator" source="#request.peoples" var="peoples2"></s:sort>
<s:iterator value="#attr.peoples2">
	${name }-${age }<br>
</s:iterator>
```

	s:sort
	AAA-111
	BBB-222
	CCC-333
	DDD-444
	EEE-555

## date标签

| 名称   | 必填  | 缺省值 | 类型    | 描述                                     |
| :--    | :--   | :--    | :--     | :--                                      |
| name   | true  |        | String  | 将被排版的日期值                         |
| format | false |        | String  | 日期的格式                               |
| nice   | false | false  | Boolean | 指定是否输出日期和当前日期之间的时间差   |
| var    | false |        | String  | 用来引用被压入ValueStack栈的日期值的变量 |

> format属性值必须是java.util.SimpleDateFormat类里定义的日期、时间格式之一

```html
s:date<br>
    <s:date name="#session.date" format="yyyy-MM-dd HH:mm:ss" var="date23"></s:date>
    ${date23 }
```

	s:date
	2017-10-13 10:34:00

## iterator标签

> Iterator：标签用于对集合进行迭代，这里的集合包含List、Set和数组。  

| 名称   | 必填  | 缺省值 | 类型          | 描述                                                               |
| :--    | :--   | :--    | :--           | :--                                                                |
| status | false |        | Object/String | 如果设置此参数，一个IteratorStatus的实例将会压入每一个遍历的堆栈   |
| value  | false |        | Object/String | 要遍历的可枚举的(iteratable)数据源，或者将其放入的新列表(List)的对象 |
| id     | false |        | Object/String | 引用变量的名称，用来标识元素的id。在ui和表单中为HTML的id属性                       |

> status：可选属性，该属性指定迭代时的IteratorStatus实例。该实例包含如下几个方法：  
> 
> 		int getCount()， 	返回当前迭代了几个元素。
> 		int getIndex()， 	返回当前迭代元素的索引。
> 		boolean isEven()， 	返回当前被迭代元素的索引是否是偶数
> 		boolean isOdd()， 	返回当前被迭代元素的索引是否是奇数
> 		boolean isFirst()， 	返回当前被迭代元素是否是第一个元素。
> 		boolean isLast()， 	返回当前被迭代元素是否是最后一个元素。

举例

```html
<s:iterator value="allList">
    <s:property value="name"/><br>
</s:iterator>

<s:iterator value="allList" var="person" begin="2" end="7"  step="2">
    <s:property value="#person.name"/><s:property value="#person.age"/><br>
</s:iterator>
```

```html
<s:iterator value="allList" status="st">
    st.getCount():<s:property value="#st.count"/> &nbsp;&nbsp;
    st.getIndex():<s:property value="#st.index"/>  &nbsp;&nbsp;
    st.isEven():<s:property value="#st.even"/>&nbsp;&nbsp;
    st.isOdd():<s:property value="#st.odd"/>&nbsp;&nbsp;
    st.isFirst:<s:property value="#st.first"/>&nbsp;&nbsp;
    st.isLast():<s:property value="#st.last"/><br>
 </s:iterator>  
```

```html
<table border="1">  
   <s:iterator value="allList" var="person" status="st">
      <tr class=<s:property value="#st.even?'even':'odd'"/>  >
         <td><s:property value="#person.name"/></td>
      </tr>
   </s:iterator>  
</table>
```

## if/elseif/else标签

> if/elseif/else  基本的流程控制.‘If’标签可单独使用也可以和‘Else If’标签和(或)一个多个‘Else’一起使用 

| 名称 | 必填  | 缺省值 | 类型          | 描述                                         |
| :--  | :--   | :--    | :--           | :--                                          |
| test | true  |        | Boolean       | 决定标志里的内容是否显示的表达式             |
| id   | false |        | Object/String | 用来标识元素的id。在UI和表单中为HTML的id属性 |

```html
<s:if test="#age==23">23</s:if>
<s:elseif test="#age==21">21</s:elseif>
<s:else>都不等</s:else>
```

实例

```html
<table border="1">  
   <s:iterator value="allList" var="person">
      <tr>
         <td><s:property value="#person.name"/></td>
         <td><s:property value="#person.age"/></td>
         <td><s:if test="#person.age<24">少年</s:if>
             <s:elseif test="#person.age<26">中年</s:elseif>
             <s:else>老年</s:else>
         </td>
      </tr>
   </s:iterator>  
</table>
```

## url标签

> url:该标签用于创建url,可以通过"param"标签提供request参数.
> 	
>		value:如果不提供就用当前action,使用value后缀必须加.action  

| 名称           | 必填  | 缺省值 | 类型    | 描述                                                                                         |
| :--            | :--   | :--    | :--     | :--                                                                                          |
| action         | false |        | String  | 指定生成的url为哪个action，如果没有则使用value                                                                    |
| anchor         | false |        | String  | 指定被创建的url的链接锚点                                                                    |
| encode         | false | true   | Boolean | 是否对参数进行编码                                                                           |
| escapeAmp      | false | true   | Boolean | 是否要对“&”字符进行转义                                                                      |
| includeContext | false | true   | Boolean | 是否要把当前的上下文包括进来                                                                 |
| includeParams  | false | get    | String  | 指定是否包含请求参数，可以取3个值之一：one/get/all                                           |
| method         | false |        | String  | 指定action的方法，当用action属性来生成url是，如果指定该属性，url将链接到指定的action的方法上 |
| namespace      | false |        | String  | 指定url的命名空间                                                                            |
| portletMode    | false |        | String  | 指定结果页面的portlet模式                                                                    |
| protletUrlType | false |        | String  | 指定将被创建的URL是一个protlet例程，还是action URL                                           |
| schema         | false |        | String  | 指定使用什么协议：http,https                                                                 |
| value          | false |        | String  | 指定将生成的url值（如果新建URL的不是一个action的话），使用value后缀必须加.action                                         |
| var            | false |        | String  | 引用变量的名称，指定用来被压入ContextMap中的键值                                                             |
| windowState    | false |        | String  | 当用在一个portlet环境时，用来指定portlet的窗口状态                                           |

举例

```html
使用action<br>
<s:url action="ognlTagAction_test" namespace="/ognl" var="myurl">
     <s:param name="name" value="%{'张老师'}"></s:param>
     <s:param name="id" value="12"></s:param>
</s:url>

<a href='<s:property  value="#myurl" />' >xxxx</a><br>
<a href='<s:property  value="%{#myurl}" />' >xxxx</a><br>

使用value<br>
<s:url value="ognlTagAction_test.action" namespace="/ognl" var="myurl">
     <s:param name="id" value="12"></s:param>
     <s:param name="cnname" value="%{'zhang'}"></s:param>
</s:url>

<a href='<s:property  value="#myurl" />' >xxxx</a><br>
```

## a标签

a标签将呈现为一个html链接，这个标签可以接受HTML语言中的a元素所能接受的所有属性

```html
 a：标签<br>
    <s:iterator value="#request.peoples">
    <s:a href="/myTest.action?name=%{name}">${age }</s:a><br>
    </s:iterator>
```

	a：标签
	111
	444
	333
	555
	222

## ognl操作集合

| Java Code                     | OGNL expression |
| :---                          | :---            |
| list.get(0)                   | list[0]         |
| array[0]                      | array[0]        |
| (User)(list.get(0)).getName() | list[0].name    |
| array.length                  | array.length    |
| list.size()                   | list.size       |
| list.isEmpty()                | list.isEmpty    |

> Action中:
>        private List   allList = new ArrayList();  
> Jsp:
>        集合的长度:`<s:property value="allList.size"/>`

### 使用ognl操作list和数组

| Java Code                                                                                                                   | OGNL expression |
| :---                                                                                                                        | :---            |
| List list =new ArrayList(3);<br/>list.add(new Integer(1));<br/>list.add(new Integer(3));<br/>list.add(new Integer(5));<br/>return list; | \{1,3,5\}         |

```html
Jsp:
<s:iterator value="{1,2,3,4}">
	<s:property/><br>
</s:iterator>

<s:iterator value="{'s1','s2','s3','s4'}" var="s">
	<s:property value="#s"/><br>
</s:iterator>
```

### 使用ognl操作map 

> ognl用多种方式使用#号，每种是不同的.动态map对于动态单选按扭组很有用.  
> 创建map与创建list语法很相似,不同的是map前需要加"#"号.

| Java Code                                                     | OGNL expression   |
| :---                                                          | :---              |
| map.get("foo")                                                | map['foo']        |
| map.get(new Integer(1))                                       | map[1]            |
| User user=(User)map.get("userA");<br/> return user.getName(); | map['userA'].name |
| map.size()                                                    | map.size          |
| map.isEmpty()                                                 | map.isEmpty       |
| map.get("foo")                                                | map.foo           |


| Java Code                                                                                                         | OGNL expression                    |
| :---                                                                                                              | :---                               |
| Map map=new HashMap(2);<br/>map.put("foo","bar");<br/>map.put("baz","whazzit");<br/>return map;                   | #{"foo":"bar","baz":"whazzit"}     |
| Map map=new HashMap(3);<br/>map.put("1","one");<br/>map.put("2","two");<br/>map.put("3","three");<br/>return map; | #{"1":"one","2":"two","3":"three"} |

```html
<s:iterator value="#{'key01':'value01','key02':'value02'}">
	<s:property/><br>
</s:iterator>

<s:iterator value="#{'key01':'value01','key02':'value02'}">
	key=<s:property value="key"/>
	value=<s:property value="value"/><br>
</s:iterator>

<s:iterator value="#{'key01':'value01','key02':'value02'}" var="map">
	key=<s:property value="#map.key"/>
	value=<s:property value="#map.value"/><br>
</s:iterator>
```

## UI标签

* 表单标签将在 HTML 文档里被呈现为一个表单元素
* 使用表单标签的优点:
	- **表单回显**
	- 对页面进行布局和排版
* 标签的属性可以被赋值为一个静态的值或一个 OGNL 表达式. 如果在赋值时使用了一个 OGNL 表达式并把它用 **%{}** 括起来, 这个表达式将会被求值

### 表单标签的共同属性

| 属性             | Theme  | 数据类型 | 描述                                                           |
| :--              | :--    | :--      | :--                                                            |
| cssClass         | simple | String   | 定义html class属性                                             |
| cssStyle         | simple | String   | 定义html style属性                                             |
| title            | simple | String   | 定义html title属性                                             |
| disabled         | simple | String   | 定义html disabled属性                                          |
| label *          | xhtml  | String   | 定义表单元素的label                                            |
| labelPosition    | xhtml  | String   | 定义表单元素的label位置(top/left)，缺省为left                  |
| requiredposition | xhtml  | String   | 定义required标识相对的label元素的位置(left/right)，缺省为right |
| name             | simple | String   | 表单元素的name映射                                             |
| required  *      | xhtml  | Boolean  | 在label中添加(true增加，否则不增加)                            |
| tabIndex         | simple | String   | 定义html tabindex属性                                          |
| value            | simple | Object   | 定义表单元素的value                                            |

“ * ”：该属性只在没有使用 simple 主题时才可以使用

### form标签

| 名称         | 必填  | 缺省值            | 类型          | 描述                                                             |
| :--          | :--   | :--               | :--           | :--                                                              |
| onsubmit     | false |                   | Object/String | HTML onsubmit属性                                                |
| action       | false | current action    | Object/String | 设置提交的action名字，不需要“.action”后缀                        |
| target       | false |                   | Object/String | HTML表单target属性                                               |
| enctype      | false |                   | Object/String | HTML表单enctype属性                                              |
| method       | false |                   | Object/String | HTML表单method属性                                               |
| namespace    | false | current namespace | Object/String | 所提交的Action的命名空间                                         |
| validate     | false | false             | Boolean       | 是否执行客户端/远程校验，使用xhtml/ajax或者继承它们的theme时有效 |
| portletMode  | false |                   | Object/String | 表单提交后显示的portlet模式                                      |
| windowState  | false |                   | Object/String | 表单提交后要显示的window的state                                  |
| openTemplate | false |                   | Object/String | 用来输出html开始部分的模板                                       |
| theme        | false |                   | Object/String | 输出元素时使用的主题(theme)(不使用缺省的)                        |

> 默认情况下, form 标签将被呈现为一个表格形式的 HTML 表单. 嵌套在 form 标签里的输入字段
 将被呈现为一个表格行. 每个表格行由两个字段组成, 一个对应着行标, 一个对应着输入元素. 
提交按钮将被呈现为一个横跨两列单元格的行

### textfield/password/hidden标签

> textfield 标签将被呈现为一个输入文本字段  
> password 标签将被呈现为一个口令字段  
> hidden 标签将被呈现为一个不可见字段  

| 名称      | 必填  | 缺省值 | 类型          | 描述                                  |
| :--       | :--   | :--    | :--           | :--                                   |
| maxlength | false |        | Integer       | 文本输入控件可以输入字符的最大长度    |
| maxLength | false |        | Object/String | 不建议使用，建议使用maxlength属性替代 |
| readonly  | false | false  | Boolean       | 当该属性为true时，不能输入            |
| size      | false |        | Integer       | 指定可视尺寸                          |

> password 标签扩展自 textfield 标签, 多了一个 showPassword 属性.该属性是**布尔型**。默认值为 **false**, 它决定着在表单回显时是否显示输入的密码. true显示密码

### submit标签

submit 标签将呈现为一个提交按钮. 根据其 type 属性的值. 这个标签可以提供 3 种呈现效果:  

> input: `<input type=“submit” …/>`  
> button: `<input type=“button” …/>`  
> image: `<input type=“image” />`  

| 名称   | 必填  | 缺省值 | 类型   | 描述                                       |
| :--    | :--   | :--    | :--    | :--                                        |
| action | false |        | String | 设置action属性                             |
| method | false |        | String | 设置method属性                             |
| align  | false |        | String | HTML的align属性                            |
| type   | false | input  | String | submit的类型，有效值为：input,button,image |

### reset标签

reset 标签将呈现为一个重置按钮. 根据其 type 属性的值. 
	这个标签可以提供 2 种呈现效果:  
> input: `<input type="reset" …/>`  
> button: `<input type="button" …/>`

### label标签

label 标签将呈现一个 HTML 行标元素

| 名称 | 必填  | 缺省值 | 类型          | 描述          |
| :--  | :--   | :--    | :--           | :--           |
| for  | false |        | Object/String | HTML for 属性 |

### textarea标签

textarea 标签将呈现为一个 HTML 文本域元素

| 名称     | 必填  | 缺省值 | 类型          | 描述                                         |
| :--      | :--   | :--    | :--           | :--                                          |
| cols     | false |        | Integer       | 列数                                         |
| rows     | false |        | Integer       | 行数                                         |
| readonly | false | false  | Boolean       | 当该属性为true时，不能输入                   |
| wrap     | false | false  | Boolean       | 指定多行文本输入控件是否应该换行             |
| id       | false |        | Object/String | 用来标识元素的id。在ui和表单中为HTML的id属性 |

```html
<s:textarea name="personal" cols="10" rows="5" label="个人简历"></s:textarea>
```

### checkbox标签

* checkbox 标签将呈现为一个 HTML 复选框元素. 
* 与其他 HTML 输入元素类似, 当包含着一个单选框的表单被提交时, 这个单选框按钮在 HTTP 请求里增加一个请求参数. 如果某个单选框被选中了, 它的值将为 true, 但**如果该单选框未被选中, 在请求中就不会增加一个请求参数**. checkbox 标签解决了这个局限性, 它采取的办法是为单选框元素创建一个配对的不可见字段

```html
<s:checkbox name="cart" label="证件"></s:checkbox>
```

☟↓

```html
<input type="checkbox" name="cart" value="true" id="_cart" />
<input type="hidden" id="_checkbox_cart" name="_checkbox_cart" value="true" />
```

* checkbox 标签有一个 **fieldValue** 属性, 该属性指定的值将在用户提交表单时作为被选中的单选框的实际值发送到服务器. 如果没有使用 **fieldValue** 属性, 单选框的值将为 true 或 false.

| 名称       | 必填  | 缺省值 | 类型          | 描述                          |
| :--        | :--   | :--    | :--           | :--                           |
| fieldValue | false | true   | Object/String | checkbox的真实HTML的value属性 |

```html
<s:checkbox name="cart" label="证件" fieldValue="apple"></s:checkbox>
```

☟↓
> **fieldValue**中的**apple** 就对应**value**中的**apple**

```html
<input type="checkbox" name="cart" value="apple" id="_cart" />
<input type="hidden" id="_checkbox_cart" name="_checkbox_cart" value="apple" />
```

### checkboxlist标签

checkboxlist 标签将呈现一组多选框

| 名称      | 必填  | 缺省值 | 类型          | 描述                                                                                                                 |
| :--       | :--   | :--    | :--           | :--                                                                                                                  |
| list      | true  |        | Object/String | 设置的用来迭代的值，如果list是一个Map(key,value)<br/>Map的key会成为选项的"value"的参数，Map的value会成为选项的内容体 |
| listKey   | false |        | Object/String | list内含对象用来获取字段的value属性                                                                                  |
| listValue | false |        | Object/String | list内含对象用来获取字段的内容属性                                                                                   |

> checkbox 标签被映射到一个字符串数组或是一个基本类型的数组. 若它提供
的多选框一个也没有被选中，相应的属性将被赋值为一个空数组而不是空值.

#### 集合为list

```html
<s:checkboxlist name="list" list="{'Java','.Net','RoR','PHP'}" value="{'Java','PHP'}"/>
```

生成如下html代码：

```html
<input type="checkbox" name="list" value="Java" checked="checked"/><label>Java</label>
<input type="checkbox" name="list" value=".Net" /><label>.Net</label>
<input type="checkbox" name="list" value="RoR"/><label>RoR</label>
<input type="checkbox" name="list" value="PHP" checked="checked"/><label>PHP</label>
```

#### 集合为Map

```html
<s:checkboxlist name="map" list="#{1:'瑜珈用品',2:'户外用品',3:'球类',4:'自行车'}" listKey="key" listValue="value" value="{1,2,3}"/>
```

生成如下html代码：

```html
<input type="checkbox" name="map" value="1" checked="checked"/><label>瑜珈用品</label>
<input type="checkbox" name="map" value="2" checked="checked"/><label>户外用品</label>
<input type="checkbox" name="map" value="3" checked="checked"/><label>球类</label>
<input type="checkbox" name="map" value="4"/><label>自行车</label>
```

#### 集合为javabean

```java
<%
  Person person1 = new Person(1,"第一个");
  Person person2 = new Person(2,"第二个");
  List<Person> list = new ArrayList<Person>();
  list.add(person1);
  list.add(person2);
  request.setAttribute("persons",list);
%>
```

```html
<!-- id和name为Person的属性 -->
<s:checkboxlist name="beans" list="#request.persons" listKey="id" listValue="name"/>
```

生成如下html代码：

```html
<input type="checkbox" name="beans" value="1"/><label>第一个</label>
<input type="checkbox" name="beans" value="2"/><label>第二个</label>
```

### radio标签

radio 标签将呈现为一组单选按钮, 单选按钮的个数与程序员通过该标签的 list 属性提供的选项的个数相同。  
一般地, **使用 radio 标签实现 “多选一”, 对于 “真/假” 则该使用 checkbox 标签**

> 该标签的使用和checkboxlist复选框相同

```html
如果集合里存放的是javabean(id和name为Person的属性)
<s:radio name="beans" list="#request.persons" listKey="personid" listValue="name"/>

生成如下html代码：
<input type="radio" name="beans" id="beans1" value="1"/><label>第一个</label>
<input type="radio" name="beans" id="beans2" value="2"/><label>第二个</label>

如果集合为MAP
<s:radio name="map" list="#{1:'瑜珈用品',2:'户外用品',3:'球类',4:'自行车'}" listKey="key" listValue="value“ value="1"/>

生成如下html代码：
<input type="radio" name="map" id="map1" value="1"/><label for="map1">瑜珈用品</label>
<input type="radio" name="map" id="map2" value="2"/><label for="map2">户外用品</label>
<input type="radio" name="map" id="map3" value="3"/><label for="map3">球类</label>
<input type="radio" name="map" id="map4" value="4"/><label for="map4">自行车</label>

如果集合为list
<s:radio name="list" list="{'Java','.Net'}" value="'Java'"/>

生成如下html代码：
<input type="radio" name="list" checked="checked" value="Java"/><label>Java</label>
<input type="radio" name="list" value=".Net"/><label>.Net</label>
```

### select标签

select 标签将呈现一个select 元素

| 名称        | 必填  | 缺省值 | 类型          | 描述                                                                                                                                        |
| :--         | :--   | :--    | :--           | :--                                                                                                                                         |
| list        | true  |        | Object/String | 创建列表的可迭代数据源，如果该列表是一个Map(key,value)，那么Map的主键将作为选项(<option\>)的'value'属性，而该主键对应的值作为选项的文本内容 |
| listKey     | false |        | Object/String | 列表数据中的元素对象的属性，用于获取选项的值                                                                                                |
| listValue   | false |        | Object/String | 列表数据源中对象的属性，用于获取选项的文本内容                                                                                              |
| headerKey   | false |        | Object/String | 设置当用户选择了header选项时，提交的的value，如果使用该属性，不能为该属性设置空值                                                           |
| headerValue | false |        | Object/String | 列表的题头选项值                                                                                                                            |
| emptyOption | false | false  | Boolean       | 是否在题头选项后面添加一个空的(--)选项                                                                                                      |
| multiple    | false |        | Object/String | 创建一个多选列表，如果value属性指定了一个数组(正确的元素类型)，那么将预先选中的数组中指定的多个选项                                         |
| size        | false |        | Integer       | 该组件列表框的大小(显示元素个数)                                                                                                            |

#### 集合为list

```html
<s:select name="list" list="{'Java','.Net'}" value="'Java'"/>

生成如下html代码：
<select name="list" id="list">
    <option value="Java" selected="selected">Java</option>
    <option value=".Net">.Net</option>
</select>

```

#### 集合为map

```html
<s:select name="map" list="#{1:'瑜珈用品',2:'户外用品',3:'球类',4:'自行车'}" listKey="key" listValue="value" value="1"/>

生成如下html代码：
<select name="map" id="map">
    <option value="1" selected="selected">瑜珈用品</option>
    <option value="2">户外用品</option>
    <option value="3">球类</option>
    <option value="4">自行车</option>
</select>

```

#### 集合为javabean

```html
<s:select name="beans" list="#request.persons" listKey="personid" listValue="name"/>

生成如下html代码：
<select name="beans" id="beans">
    <option value="1">第一个</option>
    <option value="2">第二个</option>
</select>
```

## 主题

为了让所有的 UI 标签能够产生同样的视觉效果而归集到一起的一组模板. 即**风格
相近的模板被打包为一个主题**

1. simple: 把 UI 标签翻译成最简单的 HTML 对应元素, 而且会忽视行标属性
2. xhtml: xhtml 是默认的主题. 这个主题的模板通过使用一个布局表格提供了一
      种自动化的排版机制. (默认值)
3. css_xhtml: 这个主题里的模板与 xhtml 主题里的模板很相似, 但它们将使用 css
      来进行布局和排版
4. ajax: 这个主题里的模板以 xhtml 主题里德模板为基础, 但增加了一些 Ajax 功能

### 修改主题

A、通过 UI 标签的 theme属性(只适用于当前的标签)  
    `<s:textfield name="username"  label="用户名"  theme="simple"></s:textfield>`

B、在一个表单里, 若没有给出某个 UI 标签的 theme 属性, 它将使用这个表单的主题(适用于整个form标签)  
    `<s:form  action="" method="post" namespace="/ui“    theme="simple">`

C、修改 struts.properties 文件中的 struts.ui.theme 属性. (适用整个环境)  
         `<!-- 设置ui标签的主题 -->`  
         `<constant name="struts.ui.theme" value="simple"></constant>`

优先级：A>B>C
