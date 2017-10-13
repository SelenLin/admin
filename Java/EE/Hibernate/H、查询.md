[TOC]

# 查询概述

## 实体类及映射配置

> Dept.java

```java
import java.util.HashSet;
import java.util.Set;

public class Dept {

	private int deptId;
	private String deptName;
	// 【一对多】 部门对应的多个员工
	private Set<Employee> emps = new HashSet<Employee>();
}
```

> Dept.hbm.xml

```xml
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC 
	"-//Hibernate/Hibernate Mapping DTD 3.0//EN"
	"http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">

<hibernate-mapping package="com.theliang.query">

	<class name="Dept" table="t_dept" >
		<id name="deptId">
			<generator class="native"></generator>
		</id>
		<property name="deptName" length="20"></property>
		 <set name="emps">
		 	 <key column="dept_id"></key>
		 	 <one-to-many class="Employee"/>
		 </set>
	</class>
</hibernate-mapping>
```

> Employee.java

```java
public class Employee {

	private int empId;
	private String empName;
	private double salary;
	// 【多对一】员工与部门
	private Dept dept;;
}
```

> Employee.hbm.xml

```xml
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC 
	"-//Hibernate/Hibernate Mapping DTD 3.0//EN"
	"http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">

<hibernate-mapping package="com.theliang.query">
	
	<class name="Employee" table="t_employee">
		<id name="empId">
			<generator class="native"></generator>
		</id>	
		<property name="empName" length="20"></property>
		<property name="salary" type="double"></property>
		
		<many-to-one name="dept" column="dept_id" class="Dept"></many-to-one>

	</class>
</hibernate-mapping>
```

## Get/load 主键查询

```java
public void get_load(){
	Session session=sf.openSession();
	session.beginTransaction();
	
		Dept dept =  (Dept) session.get(Dept.class, 1);
		//懒加载
		Dept dept =  (Dept) session.load(Dept.class, 1);

	session.getTransaction().commit();
	session.close();
}
```

> 详见：懒加载－load和get的区别

## 对象导航查询

```java
public void get(){
	Session session=sf.openSession();
	session.beginTransaction();

		Dept dept=(Dept)session.get(Dept.class,1);
		System.out.println(dept.getDeptName());
		System.out.println(dept.getEmps());

	session.getTransaction().commit();
	session.close();
}
```

## HQL查询

> 注意：使用hql查询的时候 `auto-import="true"` 要设置true，如果是false，写HQL的时候，要指定类的全名

```java
public void get(){
	Session session=sf.openSession();
	session.beginTransaction();

		Query q=session.createQuery("from Dept");
		System.out.println(q.list());

	session.getTransaction().commit();
	session.close();
}
```

### a.查询全部列

```java
public void get(){
	Session session=sf.openSession();
	session.beginTransaction();

		Query q = session.createQuery("from Dept");  //OK
		Query q = session.createQuery("select * from Dept");  //N0 OK, 错误，不支持 ‘*’
		Query q = session.createQuery("select d from Dept d");  // 使用别名 OK
		System.out.println(q.list());

	session.getTransaction().commit();
	session.close();
}
```

### b.查询指定的列  【返回对象数据Object[] 】

```java
public void get(){
	Session session=sf.openSession();
	session.beginTransaction();

		Query q = session.createQuery("select d.deptId,d.deptName from Dept d");
		System.out.println(q.list());

	session.getTransaction().commit();
	session.close();
}
```

### c. 查询指定的列, 自动封装为对象  【必须要提供带参数构造器】

```java
public void get(){
	Session session=sf.openSession();
	session.beginTransaction();

		Query q = session.createQuery("select new Dept(d.deptId,d.deptName) from Dept d");
		System.out.println(q.list());

	session.getTransaction().commit();
	session.close();
}
```

> 自动封闭为Dept对象，必须在对应实体类中提供带对应参数构造器

```java
import java.util.HashSet;
import java.util.Set;

public class Dept {

	private int deptId;
	private String deptName;
	// 【一对多】 部门对应的多个员工
	private Set<Employee> emps = new HashSet<Employee>();

	//不带参数构造器
	public Dept() {
		super();
	}
	//带Id、Name参数构造器
	public Dept(int deptId, String deptName) {
		super();
		this.deptId = deptId;
		this.deptName = deptName;
	}
}
```

### d. 条件查询: 一个条件/多个条件and or/between and/模糊查询

#### 条件查询：占位符

```java
public void get(){
	Session session=sf.openSession();
	session.beginTransaction();

		Query q = session.createQuery("from Dept d where deptName=?");
		q.setString(0, "财务部");
		//也可以使用这个
//		q.setParameter(0, "财务部");

		System.out.println(q.list());

	session.getTransaction().commit();
	session.close();
}
```

#### 条件查询：命名参数

```java
public void get(){
	Session session=sf.openSession();
	session.beginTransaction();

		Query q = session.createQuery("from Dept d where deptId=:myId or deptName= :name");
		q.setParameter("myId", 12);
		q.setParameter("name", "财务部");
		System.out.println(q.list());

	session.getTransaction().commit();
	session.close();
}
```

#### 范围 between and

```java
public void get(){
	Session session=sf.openSession();
	session.beginTransaction();

		Query q = session.createQuery("from Dept d where deptId between ? and ?");
		q.setParameter(0, 1);
		q.setParameter(1, 20);
		System.out.println(q.list());

	session.getTransaction().commit();
	session.close();
}
```

#### 模糊 like

```java
public void get(){
	Session session=sf.openSession();
	session.beginTransaction();

		Query q = session.createQuery("from Dept d where deptName like ?");
		q.setString(0, "%部%");
		System.out.println(q.list());

	session.getTransaction().commit();
	session.close();
}
```

### e. 聚合函数统计

> 拓展：另见 list和iterator的区别   

> MySql语句：  

```mysql
select count(*) from t_Dept; -- 统计总记录
select count(1) from t_Dept; -- 统计总记录，效率相对高

select count(deptName) from t_Dept -- 忽略NULL值【聚合函数统计都会忽略NULL值】
```

```java
public void get(){
	Session session=sf.openSession();
	session.beginTransaction();

		Query q = session.createQuery("select count(*) from Dept");
		Long num = (Long) q.uniqueResult();
		System.out.println(num);

	session.getTransaction().commit();
	session.close();
}
```

### f. 分组查询

```mysql
-- 数据库写法: 统计t_employee表中每个部门的人数
select dept_id,count(*) from t_employee group by dept_id;
```

```java
//HQL写法
public void get(){
	Session session=sf.openSession();
	session.beginTransaction();

		Query q = session.createQuery("select e.dept, count(*) from Employee e group by e.dept");
		System.out.println(q.list());

	session.getTransaction().commit();
	session.close();
}
```

### g. 连接查询

#### 内连接

```mysql
-- 两种内连接查询方式
SELECT e.empName,d.deptName FROM t_dept d,t_employee e WHERE d.deptId=e.dept_id

SELECT e.empName,d.deptName FROM t_dept d INNER JOIN t_employee e ON d.deptId=e.dept_id
```

> 测试
>> 映射已经配置好了关系，关联的时候，直接写对象的属性即可

```java
public void join(){
	Session session=sf.openSession();
	session.beginTransaction();

		Query q = session.createQuery("FROM Dept d INNER JOIN d.emps e");
		System.out.println(q.list());

	session.getTransaction().commit();
	session.close();
}
```

> Hibernate执行的代码

```mysql 
-- Hibernate执行的代码: 
    select
        dept.deptId as deptId0_0_,
        emps.empId as empId1_1_,
        dept.deptName as deptName0_0_,
        emps.empName as empName1_1_,
        emps.salary as salary1_1_,
        emps.dept_id as dept4_1_1_ 
    from
        t_dept dept
    inner join
        t_employee emps
            on dept.deptId=emps.dept_id
```


#### 左外连接

```mysql
SELECT e.empName,d.deptName FROM t_dept d LEFT JOIN t_employee e ON d.deptId=e.dept_id
```

> 测试

```java
public void join(){
	Session session=sf.openSession();
	session.beginTransaction();

		Query q = session.createQuery("FROM Dept d LEFT JOIN d.emps e");
		System.out.println(q.list());

	session.getTransaction().commit();
	session.close();
}
```

#### 右外连接[结果和左外连接一样]

```mysql
SELECT e.empName,d.deptName FROM t_employee e RIGHT JOIN t_dept d ON d.deptId=e.dept_id
```

> 测试

```java
public void join(){
	Session session=sf.openSession();
	session.beginTransaction();

		Query q = session.createQuery("FROM d.emps e RIGHT JOIN Dept d");
		System.out.println(q.list());

	session.getTransaction().commit();
	session.close();
}
```

### g. 连接查询 - 迫切连接

> `frech` 使用fetch, 会把右表的数据，填充到左表对象中使用fetch, 会把右表的数据，填充到左表对象中

> 迫切内连接

```java
Query q = session.createQuery("FROM d.emps e INNER JOIN fetch Dept d");
```

> 迫切左外连接

```java
Query q = session.createQuery("FROM d.emps e LEFT JOIN fetch Dept d");
```

### HQL查询优化

```java
public void hql_other() {
	Session session = sf.openSession();
	session.beginTransaction();
	// HQL写死
	//Query q = session.createQuery("from Dept d where deptId < 10 ");
	
	// HQL 放到映射文件中
	Query q = session.getNamedQuery("getAllDept");
	q.setParameter(0, 10);
	System.out.println(q.list());
	
	session.getTransaction().commit();
	session.close();
}
```

> 如果要执行上面的代码，则需要在映射配置中修改如下：Dept.hbm.xml

```xml
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC 
	"-//Hibernate/Hibernate Mapping DTD 3.0//EN"
	"http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">

<hibernate-mapping package="com.theliang.query">

	<class name="Dept" table="t_dept" >
		<id name="deptId">
			<generator class="native"></generator>
		</id>	
		<property name="deptName" length="20"></property>
		 <set name="emps">
		 	 <key column="dept_id"></key>
		 	 <one-to-many class="Employee"/>
		 </set>
	</class>

<!-- **********************存放sql语句********************** -->
	<query name="getAllDept">
	<!-- 两种写法
			一种直接写查询语句，特殊符号需要转义
			另一种使用
	-->

		from Dept d where deptId &lt; ?

		<![CDATA[
			from Dept d where deptId < ?
		]]>	
	</query>
<!--*********************************************************-->
</hibernate-mapping>
```



## Criteria 查询，   完全面向对象的查询（Query By Criteria  ,QBC）  

```java
public void criteria() {
	Session session = sf.openSession();
	session.beginTransaction();
	
	Criteria criteria = session.createCriteria(Employee.class);
	// 构建条件
	criteria.add(Restrictions.eq("empId", 12));
	//相同的功能：
//	criteria.add(Restrictions.idEq(12));  // 主键查询
	
	System.out.println(criteria.list());
	
	session.getTransaction().commit();
	session.close();
}
```

## SQLQuery， 本地SQL查询

```java
public void sql() {
	Session session = sf.openSession();
	session.beginTransaction();
	
	SQLQuery q = session.createSQLQuery("SELECT * FROM t_Dept limit 5;")
		.addEntity(Dept.class);  // 也可以自动封装
	System.out.println(q.list());
	
	session.getTransaction().commit();
	session.close();
}
```

## 分页查询

```java
public void all() {
	Session session = sf.openSession();
	session.beginTransaction();
	
	 Query q = session.createQuery("from Employee");
	 
	 /**
	 *	间接查询总记录数
	 */
	 // 从记录数
	 ScrollableResults scroll = q.scroll();  // 得到滚动的结果集
	 scroll.last();							//  滚动到最后一行
	 int totalCount = scroll.getRowNumber() + 1;// 得到滚到的记录数，即总记录数
	 

	 // 设置分页参数
	 q.setFirstResult(0);
	 q.setMaxResults(3);
	 
	 // 查询
	 System.out.println(q.list());
	 System.out.println("总记录数：" + totalCount);
	
	session.getTransaction().commit();
	session.close();
}
```

