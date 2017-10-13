[TOC]

# SpringIOC容器

## 创建对象

SpringIOC容器，是spring核心内容
作用： 创建对象 & 处理对象的依赖关系

### IOC容器创建对象方式

1. 调用无参数构造器
2. 带参数构造器
3. 工厂创建对象
	* 工厂类，静态方法创建对象
	* 工厂类，非静态方法创建对象

> 配置(公共部分) bean.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd">

    <!-- ###############对象创建############### -->
	
	<!-- 以下的配置均写入此处，其它因一致就省略了-->
</beans>  
```

> 实体类 User.java

```java
package com.theliang.b_create_obj;
public class User {

	private int id;
	private String name;
	
	public User() {
		super();
		System.out.println("------User对象创建【无参数构造器】------");
	}
	public User(int id, String name) {
		System.out.println("-----User对象创建【带参数构造器】--------");
		this.id = id;
		this.name = name;
	}
	@Override
	public String toString() {
		return "User [id=" + id + ", name=" + name + "]";
	}
	public void init_user() {
		System.out.println("创建对象之后，初始化");
	}
	public void destroy_user() {
		System.out.println("IOC容器销毁，user对象回收!");
	}
	// set get ...
}
```

> 测试方法类

```java
// 测试：对象创建
public void testIOC() throws Exception {
	// 创建IOC容器对象
	ApplicationContext ac=new ClassPathXmlApplicationContext("com/theliang/b_create_obj/bean.xml");
	// 获取容器中的对象
	User user=(User) ac.getBean("user");
	
	System.out.println(user);
}
```

#### 1.调用无参数构造器

> 配置 bean.xml

```xml
<!-- 1. 默认无参数构造器  -->
<bean id="user" class="com.theliang.b_create_obj.User"></bean>
```

#### 2.带参数构造器

> 配置 bean.xml

```xml
<!--***********  方式一：直接使用构造器  **************-->
<bean id="user" class="com.theliang.b_create_obj.User">
	<constructor-arg value="100"></constructor-arg>
	<constructor-arg value="Andy"></constructor-arg>
</bean>

<!--***********  方式二：通过角标传入参数  **************-->
<bean id="user" class="com.theliang.b_create_obj.User">
	<constructor-arg index="0" type="int" value="100"></constructor-arg>
	<constructor-arg index="1" type="java.lang.String" value="Andy"></constructor-arg>
</bean>

<!--***********  方式三：引用参数使用有参构造  **************-->
<!-- 定义一个字符串，值是"Jack" ;  String s = new String("jack")-->
<bean id="str" class="java.lang.String">
	<constructor-arg value="Jacks"></constructor-arg>
</bean>
<bean id="user" class="com.theliang.b_create_obj.User">
	<constructor-arg index="0" type="int" value="100"></constructor-arg>
	<constructor-arg index="1" type="java.lang.String" ref="str"></constructor-arg>
</bean>
```

#### 3. 工厂创建对象

> 工厂类方法 ObjectFactory.java

```java
package com.theliang.b_create_obj;
//工厂，创建对象
public class ObjectFactory {

	// 实例方法创建对象
	public User getInstance() {
		return new User(100,"工厂：调用实例方法");
	}
	// 静态方法创建对象
	public static User getStaticInstance() {
		return new User(101,"工厂：调用静态方法");
	}
}
```

##### 静态方法创建对象

```xml
<!-- 先创建工厂 -->
<bean id="factory1" class="com.theliang.b_create_obj.ObjectFactory"></bean>
<!-- 在创建user对象，用factory方的实例方法 -->
<bean id="user" factory-bean="factory1" factory-method="getInstance"></bean>
```

##### 非静态方法创建对象

```xml
<!-- 
	class 指定的就是工厂类型
	factory-method  一定是工厂里面的“静态方法”
-->
<bean id="user" class="com.theliang.b_create_obj.ObjectFactory" factory-method="getStaticInstance"></bean>
```

## 对象依赖关系

Spring中，如何给对象的属性赋值？【DI, 依赖注入】

1. 通过构造函数
2. 通过set方法给属性注入值
3. p名称空间
4. 自动装配(了解)
5. 注解

> 测试类

```java
public void testIOC() throws Exception {
	// 创建IOC容器对象
	ApplicationContext ac=new ClassPathXmlApplicationContext("com/theliang/c_property/bean.xml");
	// 获取容器中的对象
	User user=(User) ac.getBean("user");
	System.out.println(user);
}
```

### 1.通过构造函数【内部bean】

> 和带参数构造器类似

```xml
<!--  1) 通过构造函数 -->
<bean id="user" class="com.theliang.c_property.User">
	<constructor-arg value="100"></constructor-arg>
	<constructor-arg value="Tom"></constructor-arg>
</bean>
```

### 2.(常用)Set方法注入值

```xml
<!-- 2) 通过set方法给属性注入值 -->
 <bean id="user" class="com.theliang.c_property.User">
 	<property name="id" value="101"></property>
 	<property name="name" value="Jack"></property>
</bean>
```

#### 案例

> UserDao.java

```java
package com.theliang.c_property;
public class UserDao {
	public void save() {
		System.out.println("DB:保存用户");
	}
}
```

> UserService.java

```java
package com.theliang.c_property;
public class UserService {
	private UserDao userDao; // = new UserDao();
	// IOC：对象的创建交给spring的外部容器完成
	public void setUserDao(UserDao userDao) {
		this.userDao = userDao;
	}
	public void save() {
		userDao.save();
	}
}
```

> UserAction.java

```java
package com.theliang.c_property;
public class UserAction {
	// Service: springIOC容器注入
	private UserService userService;
	public void setUserService(UserService userService) {
		this.userService = userService;
	}
	public String execute() {
		userService.save();
		return null;
	}
}
```

> TestAction.java

```java
public void testAction(){
	ApplicationContext ac=new ClassPathXmlApplicationContext("com/theliang/c_property/bean.xml");
	UserAction userAction=(UserAction) ac.getBean("userAction");
	ua.execute();
}
```

### 2.1 内部bean

```xml
<bean id="userAction" class="com.theliang.c_property.UserAction">
 	<property name="userService">
 		<bean id="userService" class="com.theliang.c_property.UserService">
 			<property name="userDao">
 				<bean id="userDao" class="com.theliang.c_property.UserDao"></bean>
 			</property>
 		</bean>
 	</property>
</bean>
```

```xml
<bean id="userAction" class="com.theliang.c_property.UserAction">
 	<property name="userService">
 		<bean class="com.theliang.c_property.UserService">
 			<property name="userDao">
 				<bean class="com.theliang.c_property.UserDao"></bean>
 			</property>
 		</bean>
 	</property>
</bean>
```

### 3.p 名称空间注入属性值(优化)

> 直接注入值

```xml
<!-- 传统方式注入 -->
<bean id="user" class="com.theliang.c_property.User">
 	<property name="name" value="xxx"></property>
</bean>
```

```xml
<!-- p名称空间注入 -->
<bean id="user" class="com.theliang.c_property.User" p:name="YYY"></bean>
```

> 引用 bean

>> 优化上面实例中的配置

```xml
<bean id="userDao" class="com.theliang.c_property.UserDao"></bean>
 
<bean id="userService" class="com.theliang.c_property.UserService" p:userDao-ref="userDao"></bean>

<bean id="userAction" class="com.theliang.c_property.UserAction" p:userService-ref="userService"></bean>
```

### 4.自动装配(了解)

> Spring提供的自动装配主要是为了简化配置； 但是不利于后期的维护。(一般不推荐使用)


* 根据名称自动装配：`autowire="byName"`
自动去IOC容器中找与属性名同名的引用的对象，并自动注入

```xml
<bean id="userDao" class="com.theliang.d_auto.UserDao"></bean>
 
<bean id="userService" class="com.theliang.d_auto.UserService" autowire="byName"></bean>

<!-- 根据“名称”自动装配：userAction注入的属性，会去IOC容器中自动查找与属性同名的对象 -->
<bean id="userAction" class="com.theliang.d_auto.UserAction" autowire="byName"></bean>
```

* 也可以定义到**全局**， 这样就不用每个bean节点都去写`autowire="byName"`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd"
        
        default-autowire="byName">
<!-- 	↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑↑ 		-->

	<!-- ###############自动装配############### -->
	<bean id="userDao" class="com.theliang.d_auto.UserDao"></bean>
	<bean id="userService" class="com.theliang.d_auto.UserService"></bean>
	<bean id="userAction" class="com.theliang.d_auto.UserAction"></bean>
</beans>  
```

* 根据类型自动装配：`autowire="byType"`
必须确保改类型在IOC容器中只有一个对象；否则报错

> 报错

```xml
<bean id="userDao" class="com.theliang.d_auto.UserDao"></bean>
<bean id="userService" class="com.theliang.d_auto.UserService" autowire="byName"></bean>
<bean id="userAction" class="com.theliang.d_auto.UserAction" autowire="byType"></bean>
<!-- 报错：因为上面已经有一个该类型的对象，且使用了根据类型自动装配 -->
<bean id="userService_test" class="com.theliang.d_auto.UserService" autowire="byType"></bean>
```

> 不会报错的情况

```xml
<bean id="userDao" class="com.theliang.d_auto.UserDao"></bean>
<bean id="userService" class="com.theliang.d_auto.UserService" autowire="byName"></bean>
<bean id="userAction" class="com.theliang.d_auto.UserAction" autowire="byName"></bean>
<bean id="userService_test" class="com.theliang.d_auto.UserService" autowire="byType"></bean>
```

### 5.注解

> 此方式需要导入：`spring-aop-4.3.10.RELEASE` 包

注解方式可以简化spring的IOC容器的配置！

#### 使用注解步骤

1）先引入context名称空间  
`xmlns:context="http://www.springframework.org/schema/context"`

2）开启注解扫描  
`<context:component-scan base-package="com.theliang.e_anno2"></context:component-scan>`

> 开启注解扫描(注解和XML配置，可以一起使用)

```xml
<!-- 开启注解扫描 -->
<context:component-scan base-package="com.theliang.e_anno"></context:component-scan>

<!-- 注解和XML配置，可以一起使用 -->
<bean id="userDao" class="com.theliang.e_anno.UserDao"></bean>
```

3）使用注解
通过注解的方式，把对象加入ioc容器。  
创建对象以及处理对象依赖关系

#### 相关的注解

	@Component   指定把一个对象加入IOC容器

	@Repository   作用同@Component； 在持久层使用
	@Service      作用同@Component； 在业务逻辑层使用
	@Controller    作用同@Component； 在控制层使用 

	@Resource     属性注入

```java
package com.theliang.e_anno;
import javax.annotation.Resource;
import org.springframework.stereotype.Component;

// 把当前对象加入ioc容器
@Component("userService") //相当于bean.xml 【<bean id="userDao" class=".." />】
public class UserService {
	
// 	会从IOC容器中找userDao对象，注入到当前字段
/*

<bean id="userService" class=""> 
	<property name="userDao" ref="userDao" />    @Resource相当于这里的配置
</bean>

 */
	@Resource(name="userDao")
	private UserDao userDao;

    public void save() {
        userDao.save();
    }
}
```

另外一种精简方法

```java
package com.theliang.e_anno;
import javax.annotation.Resource;
import org.springframework.stereotype.Component;

@Component// 加入IOC容器的UserDao对象的引用名称， 默认与类名一样， 且第一个字母小写
public class UserService {
	
	@Resource//  根据类型查找 【在容器中要确保该类型只有一个变量】
	private UserDao userDao;

    public void save() {
        userDao.save();
    }
}
```

