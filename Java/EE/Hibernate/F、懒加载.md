[TOC]

# 懒加载

> 懒加载：(lazy)  
> 概念：当用到数据的时候才向数据库发出SQL查询，如果不使用对象则不会发SQL语句进行查询。这就是hibernate的懒加载特性。  
> 目的：提供程序执行效率！

## Lazy(懒加载)在hibernate何处使用
 > 
 > 1. &lt;class&gt;标签上，可以取值：true/false,(默认值为：true)
 > 
 > 2. &lt;property&gt;标签上，可以取值：true/false，需要类增强工具
 > 
 > 3.  &lt;set&gt;、&lt;list&gt;集合上，可以取值：true/false/extra,(默认值为：true)
 > 
 > 4. &lt;one-to-one&gt;、&lt;many-to-one&gt;单端关联上，可以取值：false/proxy/noproxy
 > 
 > PS：session.load()方法支持lazy，而session.get()不支持lazy; 

## Hibernate的lazy生效期

> 生效期和session一样的，session关闭，lazy失效  
> 
> hibernate支持lazy策略只有在session打开状态下有效。  
> &lt;class&gt;标签上，可以取值：true/false,(默认值为：true)  

## load和get的区别？

> get: 及时加载，只要调用get方法立刻向数据库查询  
> load:默认使用懒加载，当用到数据的时候才向数据库查询。

```java
public void get_load() {
        
    Session session = sf.openSession();
    session.beginTransaction();
    Dept dept = new Dept();
    // get： 及时查询
    dept = (Dept) session.get(Dept.class, 1);//立即执行查询
    System.out.println(dept.getDeptName());
    
    // load，默认懒加载， 及在使用数据的时候，才向数据库发送查询的sql语句！
    //如果懒加载的时候查询的数据在数据库中不存在 ，则会报错
    dept = (Dept)session.load(Dept.class, 1);
    System.out.println(dept.getDeptName());//只有执行这条语句时，才发送查询sql语句

    session.getTransaction().commit();
    session.close();
}
```

## 懒加载异常

* Session关闭后，不能使用懒加载数据！
* 如果session关闭后，使用懒加载数据报错: org.hibernate.<font color="red">**LazyInitializationException**</font>: could not initialize proxy - no Session
    - 如何解决session关闭后不能使用懒加载数据的问题？
        + 方式1： 先使用一下数据
            `dept.getDeptName();`
        + 方式2：强迫代理对象初始化
        `Hibernate.initialize(dept);`
        + 方式3：关闭懒加载
            设置`lazy='false'`;
        + 方式4： 在使用数据之后，再关闭`session.close();`！ 


```java
public void get_load() {
        
    Session session = sf.openSession();
    session.beginTransaction();
    Dept dept = new Dept();

    dept = (Dept)session.load(Dept.class, 1);
    System.out.println(dept.getDeptName());//只有执行这条语句时，才发送查询sql语句
    
    //为了在session关闭后还能获取数据：

//  方式1： 先使用一下数据
    //dept.getDeptName();

//  方式2：强迫代理对象初始化
    //Hibernate.initialize(dept);

//  方式3：关闭懒加载，在映射配置中修改
    
    session.getTransaction().commit();
    session.close();
    
    // 在这里使用
    //如果在session关闭前不使用数据，则session关闭后不能再使用，否则会报错
    System.out.println(dept.getDeptName());
}
```

## lazy 值
    true    使用懒加载
    false   关闭懒加载
    extra   在集合数据懒加载时候提升效率

在真正使用数据的时候才向数据库发送查询的sql；
如果调用集合的size()/isEmpty()方法，只是统计，不真正查询数据！

### 懒加载在集合中

```java
public void testExtra(){
    Session session = sf.openSession();
    session.beginTransaction();
    
    Dept dept=(Dept)session.get(Dept.class,2);
    System.out.println("------------");
    System.out.println(dept.getDeptName());
    
    System.out.println("------------");
    System.out.println(dept.getEmps().size());
    
    System.out.println("------------");
    System.out.println(dept.getEmps().isEmpty());
    System.out.println(dept.getEmps());
    
    session.getTransaction().commit();
    session.close();
}
```

#### 对 Dept.hbm.xml 映射配置

```xml
<!-- 集合属性默认使用懒加载 -->
<set name="emps" lazy="true">
     <key column="dept_id"></key>
     <one-to-many class="Employee"/>
</set>
```

#### `true` 输出的结果：

```
Hibernate: select dept0_.deptId as deptId0_0_, dept0_.deptName as deptName0_0_ from t_dept dept0_ where dept0_.deptId=?
------------
综合部
------------
Hibernate: select emps0_.dept_id as dept4_0_1_, emps0_.empId as empId1_, emps0_.empId as empId1_0_, emps0_.empName as empName1_0_, emps0_.salary as salary1_0_, emps0_.dept_id as dept4_1_0_ from t_employee emps0_ where emps0_.dept_id=?
2
------------
false
[com.theliang.one2Many.Employee@65e98b1c, com.theliang.one2Many.Employee@2925bf5b]
```

#### `false` 输出的结果：

```
Hibernate: select dept0_.deptId as deptId0_0_, dept0_.deptName as deptName0_0_ from t_dept dept0_ where dept0_.deptId=?
Hibernate: select emps0_.dept_id as dept4_0_1_, emps0_.empId as empId1_, emps0_.empId as empId1_0_, emps0_.empName as empName1_0_, emps0_.salary as salary1_0_, emps0_.dept_id as dept4_1_0_ from t_employee emps0_ where emps0_.dept_id=?
------------
综合部
------------
2
------------
false
[com.theliang.one2Many.Employee@65e98b1c, com.theliang.one2Many.Employee@2925bf5b]
```

#### `extra` 输出的结果：

```
Hibernate: select dept0_.deptId as deptId0_0_, dept0_.deptName as deptName0_0_ from t_dept dept0_ where dept0_.deptId=?
------------
综合部
------------
Hibernate: select count(empId) from t_employee where dept_id =?
2
------------
false
Hibernate: select emps0_.dept_id as dept4_0_1_, emps0_.empId as empId1_, emps0_.empId as empId1_0_, emps0_.empName as empName1_0_, emps0_.salary as salary1_0_, emps0_.dept_id as dept4_1_0_ from t_employee emps0_ where emps0_.dept_id=?
[com.theliang.one2Many.Employee@6ad82709, com.theliang.one2Many.Employee@1ff4931d]
```
