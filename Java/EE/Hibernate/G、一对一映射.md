[TOC]

# 一对一映射

## 一、 主键与外键不相同

> 用户实体类：User.java

```java
package cn.itcast.c_one2one;
// 用户
public class User {
	private int userId;
	private String userName;
	// 用户与身份证信息， 一对一关系
	private IdCard idCard;
	
	//set get ...
}
```

> 用户映射配置：User.hbm.xml

```xml
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC 
	"-//Hibernate/Hibernate Mapping DTD 3.0//EN"
	"http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">

<hibernate-mapping package="com.theliang.one2one">

	<class name="User" table="t_User">
		<id name="userId">
			<generator class="native"></generator>
		</id>
		<property name="userName" length="22"></property>
		<!-- 一对一映射 ，没有外键方 -->
		<one-to-one name="idCard" class="IdCard"></one-to-one>
	</class>
</hibernate-mapping>
```

> 身体证实体类：IdCard.java

```java
package com.theliang.one2One;
//身份证
public class IdCard {
	// 身份证号(主键)
	private String cardNum;// 对象唯一表示(Object Identified, OID)
	private String place; //  身份证地址
	// 身份证与用户，一对一的关系
	private User user;

	//set get ...
}
```

> 身体证映射配置：IdCard.hbm.xml

```xml
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC 
	"-//Hibernate/Hibernate Mapping DTD 3.0//EN"
	"http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">

<hibernate-mapping package="com.theliang.one2one2">

	<class name="IdCard" table="t_IdCard">
		<id name="cardNum">
			<generator class="assigned"></generator>
		</id>
		<property name="place" length="20"></property>	
		<!-- 
			一对一映射，有外键方
			unique="true"   给外键字段添加唯一约束
		 -->
		<one-to-one name="user" class="User" cascade="save-update"></one-to-one>
	</class>
</hibernate-mapping>
```

> 测试类【保存】

```java
public void save() {
		
	Session session = sf.openSession();
	session.beginTransaction();
	
	User user=new User();
	user.setUserName("Jack");
	
	IdCard idCard=new IdCard();
	idCard.setCardNum("321456XXXXXX");
	idCard.setPlace("北京xxx");
	
	//关系
	idCard.setUser(user);
	
	//保存
	session.save(idCard);
	//如果不设置级联操作，则需要执行下面代码
//	session.save(user);

	session.getTransaction().commit();
	session.close();
}
```

>　执行后会在数据库中创建 `t_idcard` 表：
>> 此时的主键为 `cardNum` ，并且将 外键 `user_id` 绑定到 `t_user`表的 `userId` 上。

```mysql
CREATE TABLE `t_idcard` (
  `cardNum` VARCHAR(255) NOT NULL,
  `place` VARCHAR(20) DEFAULT NULL,
  `user_id` INT(11) DEFAULT NULL,
  PRIMARY KEY (`cardNum`),
  KEY `FK35AB3516FFCA9BA3` (`user_id`),
  CONSTRAINT `FK35AB3516FFCA9BA3` FOREIGN KEY (`user_id`) REFERENCES `t_user` (`userId`)
) ENGINE=INNODB DEFAULT CHARSET=utf8
```

## 二、主键与外键相同

> 身体证实体类：IdCard.java

```java
package com.theliang.one2one2;
//身份证
public class IdCard {
	private int user_id;
	// 身份证号(主键)
	private String cardNum;// 对象唯一表示(Object Identified, OID)
	private String place; //  身份证地址
	// 身份证与用户，一对一的关系
	private User user;

	//set get ...
}
```

> 身体证映射配置：IdCard.hbm.xml

```xml
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC 
	"-//Hibernate/Hibernate Mapping DTD 3.0//EN"
	"http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">

<hibernate-mapping package="com.theliang.one2one2">

	<class name="IdCard" table="t_IdCard">
		<id name="user_id">
			<!-- 
				id 节点指定的是主键映射, 即user_id是主键
				主键生成方式： 
					foreign  即把别的表的主键作为当前表的主键；
					property (关键字不能修改)指定引用的对象     
							对象的全名 com..User、对象映射 com.User.hbm.xml、table(id)
			 -->
			<generator class="foreign">
				<param name="property">user</param>
			</generator>
		</id>
		<property name="cardNum" length="20"></property>
		<property name="place" length="20"></property>
		
		<!-- 
			一对一映射，有外键方
			（基于主键的映射）
			 constrained="true"  指定在主键上添加外键约束
		 -->
		<one-to-one name="user" constrained="true" class="User" cascade="save-update"></one-to-one>
	</class>
</hibernate-mapping>
```

	执行相同的测试类【保存】代码

>　执行后会在数据库中创建 `t_idcard` 表：
>> 主键和外键都为 `user_id` 

```mysql
CREATE TABLE `t_idcard` (
  `user_id` INT(11) NOT NULL,
  `cardNum` VARCHAR(20) DEFAULT NULL,
  `place` VARCHAR(20) DEFAULT NULL,
  PRIMARY KEY (`user_id`),
  KEY `FK35AB3516541511B` (`user_id`),
  CONSTRAINT `FK35AB3516541511B` FOREIGN KEY (`user_id`) REFERENCES `t_user` (`userId`)
) ENGINE=INNODB DEFAULT CHARSET=utf8
```

# 组件映射与继承映射

Java主要的类主要有两种方式

* 组合关系，组合关系对应的就是组件映射
* 继承关系，继承关系对应的就是继承映射

## 组件映射

	类组合关系的映射，也叫做组件映射！
	注意：组件类和被包含的组件类，共同映射到一张表！
	需求： 汽车与车轮
	数据库 
		T_car
			主键   汽车名称  轮子大小  个数

>　JAVABean

```java
//车
public class Car {
	private int id;
	private String name;
	// 车轮
	private Wheel wheel;

	//set get ...
}
```

```java
// 车轮
public class Wheel {
	private int count;
	private int size;

	//set get ...
}
```

> 映射配置 Car.hbm.xml

```xml
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping package="com.theliang.component">
	<class name="Car" table="t_car">

		<!--映射主键-->
		<id name="id">
			<generator class="native"></generator>
		</id>

		<!--映射普通字段-->
		<property name="name" length="20"></property>
		
		<!-- 映射组件字段 -->
		<component name="wheel">
			<property name="size"></property>
			<property name="count"></property>
		</component>			 
	</class>
</hibernate-mapping>
```

> 测试

```java
public void getSave() {
	
	Session session = sf.openSession();
	session.beginTransaction();
	
	//创建对象
	Wheel wheel = new Wheel();
	Car car = new Car();

	//设置属性
	wheel.setSize(38);
	wheel.setCount(4);
	car.setName("BMW");

	//维护关系
	car.setWheel(wheel);
	
	// 保存
	session.save(car);
	
	session.getTransaction().commit();
	session.close();
}
```

## 继承映射


	需求：动物
			猫
			猴子

### 简单继承映射

	需求：动物、猫、猴子。猫继承着动物

	传统方式继承的特点就是：有多少个子类就写多少个配置文件.

> 实体类 Animal.java

```java
// 动物类
public abstract class Animal {
	private int id;
	private String name;

	//set get ...
}
```

> 实体类 Cat.java 继承自Animal

```java
public class Cat extends Animal{
    // 抓老鼠
    private String catchMouse;

    //set get ...
}
```

> 映射文件 Car.hbm.xml
>> 简单继承的映射文件很好写，**在属性上，直接写父类的属性就可以了。**  
>> 但是也有致命的缺点：**如果子类有很多，就需要写很多的配置文件**

```xml
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC 
	"-//Hibernate/Hibernate Mapping DTD 3.0//EN"
	"http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">

<!-- 简单继承 -->
<hibernate-mapping package="com.theliang.extends1">
	
	<class name="Cat" table="t_Cat">

		<!--映射主键-->
		<id name="id">
			<generator class="native"></generator>
		</id>

		<!-- 简单继承映射： 父类属性直接写 -->
		<property name="name"></property>
		<property name="catchMouse"></property>	 
	</class>
</hibernate-mapping>
```

> 测试

```java
public void getSave() {
		
	Session session = sf.openSession();
	session.beginTransaction();
	
// 保存
//	Cat cat = new Cat();
//	cat.setName("大花猫");
//	cat.setCatchMouse("抓小老鼠");
//	session.save(cat);
	
	// 获取时候注意：当写hql查询的使用，通过父类查询必须写上类的全名
	Query q = session.createQuery("from com.theliang.extends1.Animal");
	List<Animal> list = q.list();
	System.out.println(list);
	
	session.getTransaction().commit();
	session.close();
}
```

### 继承映射

<font color="red">*需求：猫、猴子、动物*</font>

#### 所有子类映射到一张表 (1张表)

> 前面我们采用的是：**每个子类都需要写成一个配置文件，映射成一张表**…
> 
> 如果子类的结构很简单，只比父类多几个属性。就像上面的例子…我们可以**将所有的子类都映射成一张表中**
> 
> 但是呢，这样是**不符合数据库设计规范的**…..因为表中的数据**可能是猫**，**可能是猴子**…这明显是不合适的…
> 
> 由于表中可能**存在猫**，**存在猴子**，**为了区分是什么类型的**。我们需要使用**鉴别器**

*生成的数据表：*

| id  | name  | catchmouse | eatFood | type_ |
| --: | :---- | :------    | :---    | :---  |
| 1   | 猫    | 捉老鼠     | NULL    | a     |
| 2   | 猴子  | NULL       | 吃香蕉  | b     |

> 实体类 Monkey.java
>> 实体和上面雷同，只多了一个猴子的实体表

```java
public class Monkey extends Animal {
    // 吃香蕉
    private String eatBanana;

    //set get ...
}
```

> 映射文件 Animal.hbm.xml
>> 使用了subClass这个节点和鉴别器

```xml
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC 
	"-//Hibernate/Hibernate Mapping DTD 3.0//EN"
	"http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<!-- 
	继承映射， 所有的子类都映射到一张表
 -->
<hibernate-mapping package="com.theliang.extends2">
	
	<class name="Animal" table="t_animal">
		<id name="id">
			<generator class="native"></generator>
		</id>
		<!-- 指定鉴别器字段(区分不同的子类),必须跟在ID的后面 -->
		<discriminator column="type_"></discriminator>
		
		<property name="name"></property>
		
		<!-- 
				每个子类都用subclass节点映射
				注意：一定要指定鉴别器字段，否则报错！
				鉴别器字段：作用是在数据库中区别每一个子类的信息， 就是一个列
			discriminator-value="cat_"
				指定鉴别器字段,即type_字段的值
				如果不指定，默认为当前子类的全名,如：com.theliang.extends2.Cat
		 -->

		 <!-- 子类：猫  -->
		 <subclass name="Cat" discriminator-value="cat_">
		 	<property name="catchMouse"></property>
		 </subclass>
		 
		 <!-- 子类：猴子  -->
		  <subclass name="Monkey" discriminator-value="monkey_">
		  	<property name="eatBanana"></property>
		  </subclass>
	</class>
</hibernate-mapping>
```

> 测试
>> 加载的是Animal父类的映射文件。保存的是cat和monkey

```java
public void getSave() {
	Session session = sf.openSession();
	session.beginTransaction();
	
	// 保存
	Cat cat = new Cat();
	cat.setName("大花猫");
	cat.setCatchMouse("抓小老鼠");
	
	Monkey m = new Monkey();
	m.setName("猴子");
	m.setEatBanana("吃10个香蕉");
	
	// 保存
	session.save(cat);
	session.save(m);
	
	session.getTransaction().commit();
	session.close();
}
```

*生成的数据表：*

| id  | name  | catchmouse | eatFood | type_   |
| --: | :---- | :------    | :---    | :---    |
| 1   | 猫    | 捉老鼠     | NULL    | cat_    |
| 2   | 猴子  | NULL       | 吃香蕉  | monkey_ |


#### 每个类映射一张表(3张表)

> 父类和子类都各对应一张表。那么就有三张表了

> 这种结构看起来是完全面向对象，但是<b>表之间的结构会很复杂，插入一条子类的信息，需要两条SQL</b>

---

>  实体类设置和上面一样

---

> 映射配置: Animal.hbm.xml ，使用到了`<joined-subclass>`这个节点

```xml
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC 
	"-//Hibernate/Hibernate Mapping DTD 3.0//EN"
	"http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">

<!-- 
	继承映射， 每个类对应一张表(父类也对应表)
 -->
<hibernate-mapping package="com.theliang.extends3">
	
	<class name="Animal" table="t_animal">
		<id name="id">
			<generator class="native"></generator>
		</id>
		<property name="name"></property>
		
		<!-- 
			子类：猫  t_cat
			key 指定_cat表的外键字段
		-->
		<joined-subclass name="Cat" table="t_cat">
			<key column="t_animal_id"></key>
			<property name="catchMouse"></property>
		</joined-subclass>
		
		<!-- 子类：猴子  t_monkey -->
		<joined-subclass name="Monkey" table="t_monkey">
			<key column="t_animal_id"></key>
			<property name="eatBanana"></property>
		</joined-subclass>
	</class>
</hibernate-mapping>
```

> 测试

```java
public void getSave() {
	Session session = sf.openSession();
	session.beginTransaction();

	// 保存
	Cat cat = new Cat();
	cat.setName("大花猫");
	cat.setCatchMouse("抓小老鼠");
	
	Monkey m = new Monkey();
	m.setName("猴子");
	m.setEatBanana("吃10个香蕉");
	
	// 保存
	session.save(cat);
	session.save(m);
	
	session.getTransaction().commit();
	session.close();
}
```

> 总结：  
	一个映射文件，存储所有的子类； 子类父类都对应表；  
   缺点：表结构比较负责，插入一条子类信息，需要用2条sql：往父类插入、往子类插入！


#### 每个子类映射一张表， 父类不对应表(2张表)


> 每个子类映射成一张表，父类不对应表。  
> 这和我们传统方式继承是一样的。只不过在 `hbm.xml` 文件中使用了`<union-subclass>`这个节点，由于有了这个节点，我们就不需要每个子类都写一个配置文件了。

----

>  实体类设置和上面一样

---

> 映射配置: Animal.hbm.xml
>> * 想要父类不映射成数据库表，只要在class中配置为`abstract='true'`即可
>> * 使用了`union-subclass`节点，主键就不能采用自动增长策略了。我们改成UUID即可。并且对应的实体id类型要改成String


```xml
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC 
	"-//Hibernate/Hibernate Mapping DTD 3.0//EN"
	"http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<!-- 
	继承映射， 每个类对应一张表(父类不对应表)
 -->
<hibernate-mapping package="com.theliang.extends4">
	<!-- 
		 abstract="true"  指定实体类对象不对应表，即在数据库段不生成表
	 -->
	<class name="Animal" abstract="true">
		<!-- 如果用union-subclass节点，主键生成策略不能为自增长！ -->
		<id name="id">
			<generator class="uuid"></generator>
		</id>
		<property name="name"></property>
		
		<!-- 
			子类：猫  t_cat
			union-subclass  
				table 指定为表名, 表的主键即为id列
		-->
		<union-subclass name="Cat" table="t_cat">
			<property name="catchMouse"></property>
		</union-subclass>
		
		<!-- 子类：猴子  t_monkey -->
		<union-subclass name="Monkey" table="t_monkey">
			<property name="eatBanana"></property>
		</union-subclass>
	</class>
</hibernate-mapping>
```

> 测试

```java
public void getSave() {
	Session session = sf.openSession();
	session.beginTransaction();
	
	// 保存
	Cat cat = new Cat();
	cat.setName("大花猫");
	cat.setCatchMouse("抓小老鼠");
	
	Monkey m = new Monkey();
	m.setName("猴子");
	m.setEatBanana("吃10个香蕉");
	
	// 保存
	session.save(cat);
	session.save(m);
	
	session.getTransaction().commit();
	session.close();
}
```

> 总结：  
> 
> * 所有的子类都写到一个映射文件;   
> * 父类不对应表； 每个子类对应一张表

---

### 总结

由于我们的传统继承映射每个子类都对应一个配置文件，这样十分麻烦。因此`T.hbm.xml`就给出了几个节点供我们使用，分别有以下的情况：  

* 子类父类共有一张表subclass，
	- 不符合数据库设计规范
	- 需要使用鉴别器

* 子类、父类都有自己的表`joined-subclass`，那么就是三张表
	- 表的结构太过繁琐
	- 插入数据时要生成SQL至少就要两条

* 子类拥有自己的表、父类不对应表【推荐】`union-subclass`
	- 父类不对应表要使用`abstract`来修饰
	- 主键的id不能使用自增长策略，修改成UUID就好了。对应的JavaBean的id设置成String就好

