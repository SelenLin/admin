[TOC]

# 连接池

> 作用：管理连接，提升连接的利用效率！
> 常用的连接池：C3P0连接池
> 
> Hibernate 自带的也有一个连接池，且对C3P0 连接池也有支持！
> 
> Hbm 自带连接池
> 
> 只维护一个连接，比较简陋

```java

#################################
### Hibernate Connection Pool ###     
#################################

hibernate.connection.pool_size 1 		【Hbm 自带连接池： 只有一个连接】

###########################
### C3P0 Connection Pool###				【Hbm对C3P0连接池支持】
###########################

#hibernate.c3p0.max_size 2				最大连接数
#hibernate.c3p0.min_size 2				最小连接数
#hibernate.c3p0.timeout 5000			超时时间
#hibernate.c3p0.max_statements 100		最大执行的命令的个数
#hibernate.c3p0.idle_test_period 3000	空闲测试时间
#hibernate.c3p0.acquire_increment 2		连接不够用的时候， 每次增加的连接数
#hibernate.c3p0.validate false

【Hbm对C3P0连接池支持，  核心类】
 告诉hib使用的是哪一个连接池技术。
#hibernate.connection.provider_class org.hibernate.connection.C3P0ConnectionProvider

```

> hibernate.cfg.xml中的配置

```xml
<!--****************** 【连接池配置】****************** -->
<!-- 配置连接驱动管理类 -->
<property name="hibernate.connection.provider_class">org.hibernate.connection.C3P0ConnectionProvider</property>
<!-- 配置连接池参数信息 -->
<property name="hibernate.c3p0.min_size">2</property>
<property name="hibernate.c3p0.max_size">4</property>
<property name="hibernate.c3p0.timeout">5000</property>
<property name="hibernate.c3p0.max_statements">10</property>
<property name="hibernate.c3p0.idle_test_period">30000</property>
<property name="hibernate.c3p0.acquire_increment">2</property>
```

# 二级缓存

## 概述

	二级缓存：
		Hibernate提供了基于应用程序级别的缓存， 可以跨多个session，
		即不同的session都可以访问缓存数据。 这个换存也叫二级缓存。　　　

		Hibernate提供的二级缓存有默认的实现，且是一种可插配的缓存框架！
		如果用户想用二级缓存，只需要在hibernate.cfg.xml中配置即可； 不想用，直接移除，不影响代码。　　　

		如果用户觉得hibernate提供的框架框架不好用，自己可以换其他的缓存框架或自己实现缓存框架都可以。

## 使用二级缓存

二级缓存，使用步骤

> 1. 开启二级缓存
> 2. 指定缓存框架
> 3. 指定那些类加入二级缓存
> 4. 测试二级缓存！

## 二级缓存配置

> 查看hibernate.properties配置文件，二级缓存如何配置？

```java
##########################
### Second-level Cache ###
##########################

#hibernate.cache.use_second_level_cache false【二级缓存默认不开启，需要手动开启】
#hibernate.cache.use_query_cache true      【开启查询缓存】

## choose a cache implementation		【二级缓存框架的实现】

#hibernate.cache.provider_class org.hibernate.cache.EhCacheProvider
#hibernate.cache.provider_class org.hibernate.cache.EmptyCacheProvider
hibernate.cache.provider_class org.hibernate.cache.HashtableCacheProvider 默认实现，其它的需要导包
#hibernate.cache.provider_class org.hibernate.cache.TreeCacheProvider
#hibernate.cache.provider_class org.hibernate.cache.OSCacheProvider
#hibernate.cache.provider_class org.hibernate.cache.SwarmCacheProvider
```

## 缓存策略

```xml
<class-cache usage="read-only"/> 				放入二级缓存的对象，只读; 
<class-cache usage="nonstrict-read-write"/> 	非严格的读写
<class-cache usage="read-write"/> 				读写； 放入二级缓存的对象可以读、写；
<class-cache usage="transactional"/> 			(基于事务的策略)
```

> hibernate.cfg.xml中的配置

```xml
<!--****************** 【二级缓存配置】****************** -->
<!-- a.  开启二级缓存 -->
<property name="hibernate.cache.use_second_level_cache">true</property>
<!-- b. 指定使用哪一个缓存框架(默认提供的) -->
<property name="hibernate.cache.provider_class">org.hibernate.cache.HashtableCacheProvider</property>
<!-- c. 指定哪一些类，需要加入二级缓存  -->
<class-cache usage="read-only" class="com.theliang.second_cache.Dept"/>
```

> 测试二级缓存

```java
// 没有/有用 二级缓存
public void testCache() {
		Session session1 = sf.openSession();
		session1.beginTransaction();
		// a. 查询一次
		Dept dept = (Dept) session1.get(Dept.class, 2);
		
		session1.getTransaction().commit();
		session1.close();
		
		System.out.println("--------------------------");

		// 第二个session
		Session session2 = sf.openSession();
		session2.beginTransaction();
		// a. 查询一次
		dept = (Dept) session2.get(Dept.class, 2);
		
		session2.getTransaction().commit();
		session2.close();
	}
/**开启二级缓存前：
Hibernate: select dept0_.deptId as deptId0_0_, dept0_.deptName as deptName0_0_ from t_dept dept0_ where dept0_.deptId=?
--------------------------
Hibernate: select dept0_.deptId as deptId0_0_, dept0_.deptName as deptName0_0_ from t_dept dept0_ where dept0_.deptId=?
*/
	
/**开启二级缓存后：
Hibernate: select dept0_.deptId as deptId0_0_, dept0_.deptName as deptName0_0_ from t_dept dept0_ where dept0_.deptId=?
--------------------------
 */
```

## 集合缓存

```xml
<!-- 集合缓存[集合缓存的元素对象，也要加入二级缓存] -->
<collection-cache usage="" collection=""/>
```

> hibernate.cfg.xml中的配置

```xml
<!--****************** 【二级缓存配置】****************** -->
<!-- a.  开启二级缓存 -->
<property name="hibernate.cache.use_second_level_cache">true</property>
<!-- b. 指定使用哪一个缓存框架(默认提供的) -->
<property name="hibernate.cache.provider_class">org.hibernate.cache.HashtableCacheProvider</property>
<!-- c. 指定哪一些类，需要加入二级缓存  -->
<class-cache usage="read-only" class="com.theliang.second_cache.Dept"/>
<!-- 集合缓存[集合缓存的元素对象，也要加入二级缓存] -->
<class-cache usage="read-only" class="com.theliang.second_cache.Employee"/>
<collection-cache usage="read-only" collection="com.theliang.second_cache.Dept.emps"/>
```

> 测试二级缓存

```java
// 没有/有用 二级缓存
public void testCache() {
		Session session1 = sf.openSession();
		session1.beginTransaction();
		// a. 查询一次
		Dept dept = (Dept) session1.get(Dept.class, 2);
		
		session1.getTransaction().commit();
		session1.close();
		
		System.out.println("--------------------------");

		// 第二个session
		Session session2 = sf.openSession();
		session2.beginTransaction();
		// a. 查询一次
		dept = (Dept) session2.get(Dept.class, 2);
		
		session2.getTransaction().commit();
		session2.close();
	}
/**开启二级缓存前：
Hibernate: select dept0_.deptId as deptId0_0_, dept0_.deptName as deptName0_0_ from t_dept dept0_ where dept0_.deptId=?
Hibernate: select emps0_.dept_id as dept4_0_1_, emps0_.empId as empId1_, emps0_.empId as empId1_0_, emps0_.empName as empName1_0_, emps0_.salary as salary1_0_, emps0_.dept_id as dept4_1_0_ from t_employee emps0_ where emps0_.dept_id=?
--------------------------
Hibernate: select dept0_.deptId as deptId0_0_, dept0_.deptName as deptName0_0_ from t_dept dept0_ where dept0_.deptId=?
Hibernate: select emps0_.dept_id as dept4_0_1_, emps0_.empId as empId1_, emps0_.empId as empId1_0_, emps0_.empName as empName1_0_, emps0_.salary as salary1_0_, emps0_.dept_id as dept4_1_0_ from t_employee emps0_ where emps0_.dept_id=?
*/
	
/**开启二级缓存后：
Hibernate: select dept0_.deptId as deptId0_0_, dept0_.deptName as deptName0_0_ from t_dept dept0_ where dept0_.deptId=?
Hibernate: select emps0_.dept_id as dept4_0_1_, emps0_.empId as empId1_, emps0_.empId as empId1_0_, emps0_.empName as empName1_0_, emps0_.salary as salary1_0_, emps0_.dept_id as dept4_1_0_ from t_employee emps0_ where emps0_.dept_id=?
--------------------------

 */
```

## 查询缓存

```java
public void listCache() {
		Session session1 = sf.openSession();
		session1.beginTransaction();
		// HQL查询  【setCacheable  指定从二级缓存找，或者是放入二级缓存】
//		Query q = session1.createQuery("from Dept");//这样不会自动放入二级查询缓存的，需要手动放入
		Query q = session1.createQuery("from Dept").setCacheable(true);
		System.out.println(q.list());
		session1.getTransaction().commit();
		session1.close();
		
		System.out.println("--------------------------");
		
		Session session2 = sf.openSession();
		session2.beginTransaction();
		q = session2.createQuery("from Dept").setCacheable(true);
		System.out.println(q.list());  // 不查询数据库： 需要开启查询缓存
		session2.getTransaction().commit();
		session2.close();
	}
/**开启二级查询缓存前：
Hibernate: select dept0_.deptId as deptId0_, dept0_.deptName as deptName0_ from t_dept dept0_
Hibernate: select emps0_.dept_id as dept4_0_1_, emps0_.empId as empId1_, emps0_.empId as empId1_0_, emps0_.empName as empName1_0_, emps0_.salary as salary1_0_, emps0_.dept_id as dept4_1_0_ from t_employee emps0_ where emps0_.dept_id=?
Hibernate: select emps0_.dept_id as dept4_0_1_, emps0_.empId as empId1_, emps0_.empId as empId1_0_, emps0_.empName as empName1_0_, emps0_.salary as salary1_0_, emps0_.dept_id as dept4_1_0_ from t_employee emps0_ where emps0_.dept_id=?
[Dept [deptId=1, deptName=会计部, emps=[com.theliang.second_cache.Employee@4659191b, com.theliang.second_cache.Employee@29215f06]], Dept [deptId=2, deptName=会计1部, emps=[com.theliang.second_cache.Employee@783a467b, com.theliang.second_cache.Employee@7db12bb6]]]
--------------------------
Hibernate: select dept0_.deptId as deptId0_, dept0_.deptName as deptName0_ from t_dept dept0_
[Dept [deptId=1, deptName=会计部, emps=[com.theliang.second_cache.Employee@e350b40, com.theliang.second_cache.Employee@41a0aa7d]], Dept [deptId=2, deptName=会计1部, emps=[com.theliang.second_cache.Employee@6340e5f0, com.theliang.second_cache.Employee@2794eab6]]]
 */
	
/**开启二级查询缓存后：
Hibernate: select dept0_.deptId as deptId0_, dept0_.deptName as deptName0_ from t_dept dept0_
Hibernate: select emps0_.dept_id as dept4_0_1_, emps0_.empId as empId1_, emps0_.empId as empId1_0_, emps0_.empName as empName1_0_, emps0_.salary as salary1_0_, emps0_.dept_id as dept4_1_0_ from t_employee emps0_ where emps0_.dept_id=?
Hibernate: select emps0_.dept_id as dept4_0_1_, emps0_.empId as empId1_, emps0_.empId as empId1_0_, emps0_.empName as empName1_0_, emps0_.salary as salary1_0_, emps0_.dept_id as dept4_1_0_ from t_employee emps0_ where emps0_.dept_id=?
[Dept [deptId=1, deptName=会计部, emps=[com.theliang.second_cache.Employee@783a467b, com.theliang.second_cache.Employee@3527942a]], Dept [deptId=2, deptName=会计1部, emps=[com.theliang.second_cache.Employee@2e3967ea, com.theliang.second_cache.Employee@70e9c95d]]]
--------------------------
[Dept [deptId=1, deptName=会计部, emps=[com.theliang.second_cache.Employee@1a942c18, com.theliang.second_cache.Employee@55a147cc]], Dept [deptId=2, deptName=会计1部, emps=[com.theliang.second_cache.Employee@71ba6d4e, com.theliang.second_cache.Employee@738dc9b]]]
 */
```