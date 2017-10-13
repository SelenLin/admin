[TOC]

# 映射配置 

> 1.普通字段类型<br/>
> 2.主键映射<br/>
> 
>   * 单列主键映射
>   * 多列作为主键映射
>   
> 主键生成策略，查看api：`hibernate-distribution-3.6.0.Final\documentation\manual\zh-CN\html_single` ---> `5.1.2.2.1. Various additional generators`


* **数据库：**

> **一个表能否有多个主键？**
> &emsp;*不能。*
> 
> **为什么要设置主键？**
> 
> &emsp;*数据库存储的数据都是有效的，必须保持唯一。*
> 
> **为什么把id作为主键？**
> 
> &emsp;*因为表中通常找不到合适的列作为唯一列即主键，所以为了方法用id列，因为id是数据库系统维护可以保证唯一，所以就把这列作为主键!*
> 
> **[联合/复合主键](#_10 "跳转到复合主键")**
> 
> &emsp;*如果找不到合适的列作为主键，出来用id列以外，我们一般用联合主键，即多列的值作为一个主键，从而确保记录的唯一性。**


---

## 实例
> Entity.hbm.xml

```xml
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC 
    "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
    "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<!-- 映射文件: 映射一个实体类对象；  描述一个对象最终实现可以直接保存对象数据到数据库中。  -->
<!-- 
    package: 要映射的对象所在的包(可选,如果不指定,此文件所有的类都要指定全路径)
    auto-import 默认为true， 在写HQL的时候自动导入包名
                如果指定为false, 再写HQL的时候必须要写上类的全名；
                  如：session.createQuery("from com.theliang.hbm_config.Employee").list();
 -->
<hibernate-mapping package="com.theliang.hbm_config" auto-import="true">
    <!-- 
        class 映射某一个对象的(一般情况，一个对象写一个映射文件，即一个class节点)
        name 指定要映射的对象的类型
        table 指定对象对应的表；
                  如果没有指定表名，默认与对象名称一样 
     -->
    <class name="Employee" table="employee">
        
        <!-- 主键 ，映射-->
        <id name="empId" column="id">
            <!-- 
                主键的生成策略
                    identity    自增长(mysql,db2)
                    sequence    自增长(序列)，oracle中自增长是以序列方法实现
                    native      自增长【会根据底层数据库自增长的方式选择identity或sequence】

                            如果是mysql数据库, 采用的自增长方式是identity
                            如果是oracle数据库， 使用sequence序列的方式实现自增长
                    
                    increment   自增长(会有并发访问的问题，一般在服务器集群环境使用会存在问题。)
                    
                    assigned    指定主键生成策略为手动指定主键的值
                    uuid        指定uuid随机生成的唯一的值
                    foreign     (外键的方式， one-to-one)
             -->
            <generator class="uuid"/>
        </id>
        <!-- 
            普通字段映射
            property
                name    指定对象的属性名称
                column  指定对象属性对应的表的字段名称，如果不写默认与对象属性一致。
                length  指定字符的长度, 默认为255
                type    指定映射表的字段的类型，如果不指定会匹配属性的类型
                        java类型：         必须写全名
                        hibernate类型：    直接写类型，都是小写
        -->
        <property name="empName" column="empName" type="java.lang.String" length="20"></property>
        <property name="workDate" type="java.util.Date"></property>
        <!-- 如果列名称为数据库关键字，需要用反引号或改列名 -->
        <property name="desc" column="`desc`" type="java.lang.String"></property>
    </class>
</hibernate-mapping>
```

## 加载映射

> 通过 `hibernate.cfg.xml` 文件中加载

> 通过代码自动加载

```java
public class loadConfig{
    private static SessionFactory sf;
    static{
        //创建sessionfactory对象
        sf=new Configuration()
            .configure()            //加载主配置文件 hibernate.cfg.xml
            .addClass(T.class)      //通过此类会自动加载映射文件：T.hbm.xml
            .buildSessionFactory();
    }
}
```

## 复合主键映射

> CompositeKeys.java

```java
/**
* 复合主键类
*   必须实现Serializable
*/
public class CompositeKeys implements Serializable{
    private String userName;
    private String address;
    //get set ...
}
```

> User.java

```java
public class User {
    // 假设：名字跟地址，不会重复
    private CompositeKeys keys;
    private int age;
}
```

> User.hbm.xml

```xml
<?xml version="1.0" ?>
<!DOCTYPE hibernate-mapping PUBLIC 
    "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
    "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">

<hibernate-mapping package="com.theliang.compositeKey" auto-import="true">
    <class name="User"> 
        <!-- 复合主键映射 -->
        <composite-id name="keys">
            <key-property name="userName" type="string"></key-property>
            <key-property name="address" type="string"></key-property>
        </composite-id>
        <!-- 普通字段映射 -->
        <property name="age" type="int"></property>     
    </class>
</hibernate-mapping>
```

> test.java

```java
import java.util.Date;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.Transaction;
import org.hibernate.cfg.Configuration;
import org.junit.Test;

public class test { 
    private static SessionFactory sf;
    static  {       
        // 创建sf对象
        sf = new Configuration()
            .configure()            //加载主配置文件 hibernate.cfg.xml
            .addClass(User.class)   //会自动加载映射文件：Employee.hbm.xml
            .buildSessionFactory();
    }
    //1. 保存对象
    @Test
    public void testSave() throws Exception {
        Session session = sf.openSession();
        Transaction tx = session.beginTransaction();
        
        // 对象
        CompositeKeys keys = new CompositeKeys();
        keys.setAddress("广州");
        keys.setUserName("Jack");
        User user = new User();
        user.setAge(20);
        user.setKeys(keys);
        
        // 保存
        session.save(user);
        tx.commit();
        session.close();
    }

    //2. 获取对象
    @Test
    public void testGet() throws Exception {
        Session session = sf.openSession();
        Transaction tx = session.beginTransaction();
        
        //构建主键再查询
        CompositeKeys keys = new CompositeKeys();
        keys.setAddress("广州");
        keys.setUserName("Jack");
        
        // 主键查询
        User user = (User) session.get(User.class, keys);
        // 测试输出
        if (user != null){
            System.out.println(user.getKeys().getUserName());
            System.out.println(user.getKeys().getAddress());
            System.out.println(user.getAge());
        }       
        tx.commit();
        session.close();
    }
}
```

> 同时,还应该配置[`hibernamte.cfg.xml`](#hibernatecfgxml)文件,上面已经配置过了,不再列举 


## 关联映射(hibernate映射)

### 1. 集合映射

> User.hbm.xml

> User.java

```java
package com.theliang.collection;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;
import java.util.Set;

public class User {

    private int userId;
    private String userName;
    // 一个用户对应多个地址
    private Set<String> address;
    private List<String> addressList=new ArrayList<String>();
    //private String[] addressArray;//映射方式和List一样:<array name=""></array>
    private Map<String, String> addressMap =new HashMap<String, String>();
    // set get ...
}
```

```xml
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC 
    "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
    "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">

<hibernate-mapping package="com.theliang.collection">

    <class name="User" table="t_user">
        <id name="userId" column="id">
            <generator class="native"></generator>
        </id>
        <property name="userName"></property>
        <!-- 
            **set集合属性的映射**
                name        指定要映射的set集合的属性
                table       集合属性要映射到的表
                key         指定集合表(t_address)的外键字段
                element     指定集合表的其他字段
                    type    元素类型，一定要指定
         -->
         <set name="address" table="t_addressSet">
            <key column="uid"></key>
            <element column="address" type="string"></element>
         </set>
         
         <!-- 
            **list集合映射**
                list-index  指定的是排序列的名称 (因为要保证list集合的有序)
          -->
          <list name="addressList" table="t_addressList">
              <key column="uid"></key>
              <list-index column="idx"></list-index>
              <element column="address" type="string"></element>
          </list>
          
          <!-- 
            **map集合的映射**
                key         指定外键字段
                map-key     指定map的key 
                element     指定map的value
           -->
          <map name="addressMap" table="t_addressMap">
            <key column="uid"></key>
            <map-key column="shortName" type="string" ></map-key>
            <element column="address" type="string" ></element>
          </map>  
    </class>
</hibernate-mapping>
```

> test.java 测试类

```java
package com.theliang.collection;

import java.util.HashSet;
import java.util.Set;

import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;
import org.hibernate.classic.Session;
import org.junit.Test;

public class App {

    private static SessionFactory sf;
    static{
        sf=new Configuration().configure()
                .addClass(User.class)
                .buildSessionFactory();
    }
    //保存Set集合
    @Test
    public void testSaveSet()throws Exception {
        Session session=sf.openSession();
        session.beginTransaction();
 
        Set<String> addressSet=new HashSet<String>();
        addressSet.add("北京Set");
        addressSet.add("上海Set");
        //用户对象
        User user=new User();
        user.setUserName("小明Set");
        user.setAddress(addressSet);
        
        //保存
        session.save(user);
        
        session.getTransaction().commit();
        session.close();
    }
    
    //保存List集合
    @Test
    public void testSaveList()throws Exception {
        Session session=sf.openSession();
        session.beginTransaction();
        
        //保存用户对象
        User user=new User();
        user.setUserName("小明List");
        user.getAddressList().add("上海List");//使用应此行代码前必须在实体中new对的对象
        user.getAddressList().add("广州List");
        
        session.save(user);
        
        session.getTransaction().commit();
        session.close();
    }
    
    //保存Map集合
    @Test
    public void testSaveMap()throws Exception {
        Session session=sf.openSession();
        session.beginTransaction();
        
        //保存用户对象
        User user=new User();
        user.setUserName("小明Map");
        user.getAddressMap().put("A0001", "北京Map");
        user.getAddressMap().put("A0002", "深圳Map");
        
        session.save(user);
        
        session.getTransaction().commit();
        session.close();
    }
}
```


#### 集合数据获取(eq)

```java
public void getAdd()throws Exception{
    Session session=sf.openSession();
    session.beginTransaction();
    
    User user=(User)session.get(User.class, 1);
    System.out.println(user.getUserId());
    System.out.println(user.getUserName());
    
    // 当查询用户，同时可以获取用户关联的list集合的数据 (因为有正确映射)
    // 当使用到集合数据的使用，才向数据库发送执行的sql语句  (懒加载)
    System.out.println(user.getAddressMap());
    
    session.getTransaction().commit();
    session.close();
}
```

### 2. 一对多与多对一映射 (重点)

> 需求1：<br/>
> &emsp;部门与员工<br/>
> &emsp;&emsp;一个部门有多个员工;       【一对多】<br/>
> &emsp;&emsp;多个员工，属于一个部门    【多对一】<br/>
> 需求2：<br/>
> &emsp;项目与开发员工<br/>
> &emsp;&emsp;一个项目，有多个开发人员！<br/>
> &emsp;&emsp;一个开发人员，参与多个项目！   【多对多】<br/>

#### 一对多

> 部门实体 Dept.java

```java
package com.theliang.one2Many;

import java.util.HashSet;
import java.util.Set;

public class Dept {

    private int deptId;
    private String deptName;
    // 【一对多】 一个部门对应的多个员工
    private Set<Employee> emps=new HashSet<Employee>();

    // get set ...
}
```

> 部门对应的映射配置 Dept.hbm.xml

```xml
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC 
    "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
    "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping package="com.theliang.one2Many" auto-import="true">
    
    <class name="Dept" table="t_dept">
        <id name="deptId">
            <generator class="native"></generator>
        </id>
        <property name="deptName" length="20"></property>
        <!-- 
            一对多关联映射配置  （通过部门管理到员工）
            Dept 映射关键点：
            1.  指定 映射的集合属性：    "emps"
            2.  集合属性对应的集合表：   "t_employee"
            3.  集合表的外键字段         "t_employee. dept_id"
            4.  集合元素的类型
            
            inverse=false  set集合映射的默认值； 表示有控制权
         -->
         <set name="emps" cascade="none" table="t_employee" inverse="false">
         <!-- <set name="emps"> 也可以实现映射-->
             <key column="dept_id"></key>
             <one-to-many class="Employee"/>
         </set>
    </class>
</hibernate-mapping>
```

#### 多对一

> 员工实体 Employee.java

```java
package com.theliang.one2Many;

public class Employee {

    private int empId;
    private String empName;
    private double salary;
    // 【多对一】员工与部门
    private Dept dept;

    // get set ...
}
```

> 员工对应的映射配置 Employee.hbm.xml

```xml
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC 
    "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
    "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping package="com.theliang.one2Many" auto-import="true">
    
    <class name="Employee" table="t_employee">
        <id name="empId">
            <generator class="native"></generator>
        </id>
        
        <property name="empName" length="20"></property>        
        <property name="salary" type="double"></property>
        
        <!-- 外键字段
            多对一映射配置
            Employee 映射关键点：
            1.  映射的部门属性  ：  dept
            2.  映射的部门属性，对应的外键字段: dept_id
            3.  部门的类型
         -->
         <many-to-one name="dept" column="dept_id" class="Dept"></many-to-one>
    </class>
</hibernate-mapping>
```

> 测试类 

```java
package cn.itcast.b_one2Many;

import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;
import org.hibernate.classic.Session;
import org.junit.Test;
import cn.itcast.a_collection.User;

public class App_save {
    private static SessionFactory sf;
    static {
        sf = new Configuration()
            .configure()
            .addClass(Dept.class)       // 测试时候使用
            .addClass(Employee.class)   // 测试时候使用
            .buildSessionFactory();
    }
    // 保存， 部门方 【对‘一’的一方操作】
    @Test
    public void save1() {
        Session session = sf.openSession();
        session.beginTransaction();
        
        // 部门对象
        Dept dept = new Dept();
        dept.setDeptName("应用开发部");
        // 员工对象
        Employee emp_zs = new Employee();
        emp_zs.setEmpName("张三");
        Employee emp_ls = new Employee();
        emp_ls.setEmpName("李四");
        /*
        * 关系：【部门中添加员工】
        */
        dept.getEmps().add(emp_zs);
        dept.getEmps().add(emp_ls);

        // 保存
        session.save(emp_zs);
        session.save(emp_ls);
        session.save(dept); // 保存部门，部门下所有的员工  
        
        session.getTransaction().commit();
        session.close();
        /*
         *  结果(如果映射配置中配置：inverse='true' ，将不显示update):
         *  Hibernate: insert into t_employee (empName, salary, dept_id) values (?, ?, ?)
            Hibernate: insert into t_employee (empName, salary, dept_id) values (?, ?, ?)
            Hibernate: insert into t_dept (deptName) values (?)
            Hibernate: update t_employee set deptId=? where empId=?    维护员工引用的部门的id
            Hibernate: update t_employee set deptId=? where empId=?
         */
    }
    // 【推荐】 保存， 员工方 【对‘多’的一方操作】
    @Test
    public void save2() {
        Session session = sf.openSession();
        session.beginTransaction();
        
        // 部门对象
        Dept dept = new Dept();
        dept.setDeptName("综合部");
        // 员工对象
        Employee emp_zs = new Employee();
        emp_zs.setEmpName("张三");
        Employee emp_ls = new Employee();
        emp_ls.setEmpName("李四");
        /*
        * 关系：【员工中添加部门】
        */
        emp_zs.setDept(dept);
        emp_ls.setDept(dept);
        
        // 保存;如果先保存‘多’再保存‘一’,会出现上一方法的结果
        session.save(dept); // 先保存'一'的方法
        session.save(emp_zs);
        session.save(emp_ls);// 再保存'多'的方法，关系回自动维护(映射配置完)
        
        session.getTransaction().commit();
        session.close();
        /*
         *  结果
         *  Hibernate: insert into t_dept (deptName) values (?)
            Hibernate: insert into t_employee (empName, salary, dept_id) values (?, ?, ?)
            Hibernate: insert into t_employee (empName, salary, dept_id) values (?, ?, ?)
            少生成2条update  sql
         */
    }
}
```

> 查询数据

```java
package com.theliang.one2Many;

import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;
import org.hibernate.classic.Session;
import org.junit.Test;

public class App_get {

    private static SessionFactory sf;
    static{
        sf=new Configuration().configure()
                .addClass(Dept.class)       // 测试时候使用
                .addClass(Employee.class)   // 测试时候使用
                .buildSessionFactory();
    }
    @Test
    public void get() {
        
        Session session = sf.openSession();
        session.beginTransaction();
        
//       通过部门方，获取另外一方
        Dept dept = (Dept) session.get(Dept.class, 1);
        System.out.println(dept.getDeptName());
        System.out.println(dept.getEmps());// 懒加载
        
        // 通过员工房，获取另外一方
        Employee emp = (Employee) session.get(Employee.class, 1);
        System.out.println(emp.getEmpName());
        System.out.println(emp.getDept().getDeptName());
        
        session.getTransaction().commit();
        session.close();
    }
}
```

#### 总结

> &emsp;&emsp;在一对多与多对一的关联关系中，保存数据最好的<font color="red">**通过‘`多`’的一方来维护关系**</font>，这样可以减少update语句的生成，从而提高hibernate的执行效率！
>     
> 配置一对多与多对一，这种叫“双向关联”<br/>
>> &emsp;只配置一对多，           叫“单项一对多”<br/>
>> &emsp;只配置多对一，           叫“单项多对一”<br/>
> 
> 注意：
>     配置了哪一方，哪一方才有维护关联关系的权限！

### 3. inverse/cascade/lazy

#### Inverse 属性

> &emsp;inverse属性用于指示本方是否参与维护关系，设为`true`时不维护，设为`false`时维护。此处的关系是指关联两张表的在维护关联关系的时候起作用。<br/>
> &emsp;外键或者关系表字段。本属性一般设置于一对多关系中的一端，并且设置为`false`，因为若由一端负责维护，每次更新完一端数据，都会去寻找于一端有关系的多段表中的行，并更新其外键字段。而由多端维护时，由于一端对象是多端对象的属性字段，所以，每次更新多端后提交数据，都会自动更新该字段（若有更新时），这样比较方便。


    Inverse , 控制反转。
    Inverse =   false       不反转；   当前方有控制权
                true        控制反转； 当前方没有控制权


##### 维护关联关系中，是否设置inverse属性


```xml
<set name="emps" table="t_employee" inverse="true">
     <key column="dept_id"></key>
     <one-to-many class="Employee"/>
 </set>
```

###### 1. 对保存数据
    
> 有影响

> &emsp;如果设置控制反转,即 `inverse=true` ，然后通过部门方维护关联关系。在保存部门的时候，同时保存员工， 数据会保存，但关联关系不会维护。即员工外键字段为 `NULL` 。同时控制台也不会出现如 `2. 一对多与多对一映射 (重点) 中的测试类`


```java
// 1. 保存数据 
@Test
public void save() {
    Session session = sf.openSession();
    session.beginTransaction();
    
    // 部门对象
    Dept dept = new Dept();
    dept.setDeptName("应用开发部");
    // 员工对象
    Employee emp_zs = new Employee();
    emp_zs.setEmpName("张三");
    Employee emp_ls = new Employee();
    emp_ls.setEmpName("李四");
    // 关系
    dept.getEmps().add(emp_zs);
    dept.getEmps().add(emp_ls);  // inverse=true,  不会设置关联。
                                //                 此时的关联应该通过员工方维护。
    
    // 保存
    session.save(emp_zs);
    session.save(emp_ls);
    session.save(dept); // 保存部门，部门下所有的员工  
    
    session.getTransaction().commit();
    session.close();
}
```

###### 2. 对获取数据

> 无影响。只要配置好了一对多和多对一的关系

```java
//是否设置inverse，对获取数据的影响？   无
public void getData() {
    Session session = sf.openSession();
    session.beginTransaction();
    
    Dept dept = (Dept) session.get(Dept.class, 1);
    System.out.println(dept.getDeptName());
    // 对于没有控制权保存的数据，获取后的数据为null
    System.out.println(dept.getEmps());
    
    session.getTransaction().commit();
    session.close();
}
```

###### 3. 对解除关联关系？

> 有

```java
// 是否设置inverse，对解除关联关系影响？
// inverse=false，  可以解除关联
// inverse=true，  当前方(部门)没有控制权，不能解除关联关系(不会生成update语句,也不会报错)
// 
@Test
public void removeRelation() {
    Session session = sf.openSession();
    session.beginTransaction();
    
    // 获取部门
    Dept dept = (Dept) session.get(Dept.class, 2);
    // 解除关系
    dept.getEmps().clear();
    
    session.getTransaction().commit();
    session.close();
}
```

###### 4. 对删除数据对关联关系的影响？

```java
// inverse=false, 有控制权， 可以删除。先清空外键引用，再删除数据。
// inverse=true,  没有控制权: 如果删除的记录有被外键引用，会报错，违反主外键引用约束！
//                           如果删除的记录没有被引用，可以直接删除。
@Test
public void deleteData() {
    Session session = sf.openSession();
    session.beginTransaction();
    
    // 查询部门
    Dept dept = (Dept) session.get(Dept.class, 8);
    session.delete(dept);
    
    session.getTransaction().commit();
    session.close();
}
```

#### Cascade 属性

> `cascade`表示级联操作，即两个实体间存在级联关系（一个类是另一个类中的属性）时，当保存、更新或删除一个实体时，是否对关联的实体做出相应操作（数据库操作）

```
cascade  表示级联操作     【可以设置到一的一方或多的一方】
    none                不级联操作， 默认值
    save-update         级联保存或更新
    delete              级联删除
    save-update,delete  级联保存、更新、删除
    all                 同上。级联保存、更新、删除
```

> 在T.hbm.xml配置文件中如下配置

```xml
<set name="emps" table="t_employee" cascade="all">
     <key column="dept_id"></key>
     <one-to-many class="Employee"/>
 </set>
```

> 级联保存【save-update】
> <br/>*这样保存dept的时候，会自动在数据库中添加两个新员工*

```java
// 级联保存
@Test
public void save() {
    Session session = sf.openSession();
    session.beginTransaction();
    
    // 部门对象
    Dept dept = new Dept();
    dept.setDeptName("财务部");
    // 员工对象
    Employee emp_zs = new Employee();
    emp_zs.setEmpName("张三");
    Employee emp_ls = new Employee();
    emp_ls.setEmpName("李四");
    // 关系
    dept.getEmps().add(emp_zs);
    dept.getEmps().add(emp_ls);  
    
    // 保存
//  session.save(emp_zs);
//  session.save(emp_ls);

    //保存部门后，自动保存部门下所有的员工
    session.save(dept); // 需要设置级联保存“cascade="save-update"”；
    
    session.getTransaction().commit();
    session.close();
}
```


> 级联删除【delete】
> <br/>*此时此classes包含的所以的student都一并删除了。注意在如果不设置级联删除则无法删除被外键引用的对象。*

```java
// 级联删除
@Test
public void delete() {
    Session session = sf.openSession();
    session.beginTransaction();
    
    Dept dept = (Dept) session.get(Dept.class,1);
    session.delete(dept); // 级联删除,同时也会删除外键对应的表
    
    session.getTransaction().commit();
    session.close();
}
```

<font size="5">*说明：*</font>

> * 在hibernate中，通过`session.save`方法保存一个持久化对象这种方式称为显示保存。
> * 在hibernate中，通过级联的方式来操作一个对象，这样的方式称为隐式操作。
> * 对`employee`对象进行了隐式的保存操作，是因为`employee`是一个临时状态的对象，在数据库中没有对应的记录，所以应该对`employee`执行insert语句

#### Lazy 属性

> 另见：F、懒加载

### 4. 多对多映射

```
需求：项目与开发人员
      Project   Developer
      电商系统
           曹吉
           王春
       OA系统
            王春
            老张
```

> 项目实体：Developer.java

```java
package com.theliang.many2Many;

import java.util.HashSet;
import java.util.Set;
// 开发人员
public class Developer {

    private int d_id;
    private String d_name;
    // 开发人员，参数的多个项目
    private Set<Project> projects = new HashSet<Project>();

    //set get ...
}
```

> Developer.hbm.xml

```xml
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC 
    "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
    "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">

<hibernate-mapping package="com.theliang.many2Many" auto-import="true">
    
    <class name="Project" table="t_project">
        <id name="prj_id">
            <generator class="native"></generator>
        </id>
        <property name="prj_name" length="20"></property>
        <!-- 
            多对多映射:
            1.  映射的集合属性： “developers”
            2.  集合属性，对应的中间表： “t_relation”
            3. 外键字段:  prjId
            4. 外键字段，对应的中间表字段:  did
            5.   集合属性元素的类型
         -->
        <set name="developers" table="t_relation" inverse="false">
            <key column="prjId"></key>
            <many-to-many column="did" class="Developer"></many-to-many>
        </set>
    </class>
</hibernate-mapping>
```

> 项目实体 Project.java

```java
package com.theliang.many2Many;

import java.util.HashSet;
import java.util.Set;
//项目
public class Project {

    private int prj_id;
    private String prj_name;
    // 项目下的多个员工
    private Set<Developer> developers=new HashSet<Developer>();

    //set get ...
}
```

> Project.hbm.xml

```xml
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC 
    "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
    "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">

<hibernate-mapping package="com.theliang.many2Many" auto-import="true">
    
    <class name="Project" table="t_project">
        <id name="prj_id">
            <generator class="native"></generator>
        </id>
        <property name="prj_name" length="20"></property>
        <!-- 
            多对多映射:
            1.  映射的集合属性： “developers”
            2.  集合属性，对应的中间表： “t_relation”
            3. 外键字段:  prjId
            4. 外键字段，对应的中间表字段:  did
            5.   集合属性元素的类型
         -->
        <set name="developers" table="t_relation" inverse="true">
            <key column="prjId"></key>
            <many-to-many column="did" class="Developer"></many-to-many>
        </set>
    </class>
</hibernate-mapping>
```


#### 测试类【保存数据】

```java
package com.theliang.many2Many;

import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;
import org.hibernate.classic.Session;
import org.junit.Test;

public class App {

    private static SessionFactory sf;
    static{
        sf=new Configuration().configure()
                .addClass(Developer.class)
                .addClass(Project.class)
                .buildSessionFactory();
    }
    
    //多对多,数据的保存
    //1. 设置inverse属性，对保存数据影响?
    // 有影响。
    // inverse=false ，有控制权，可以维护关联关系； 保存数据的时候会把对象关系插入中间表；
    // inverse=true,  没有控制权， 不会往中间表插入数据。
    @Test
    public void save()throws Exception{
        Session session=sf.openSession();
        session.beginTransaction();
        
        /*
         * 模拟数据： 
            电商系统（曹吉，王春）
            OA系统（王春，老张）
         */
        
        // 创建项目对象
        Project prj_ds = new Project();
        prj_ds.setPrj_name("电商系统");
        Project prj_oa = new Project();
        prj_oa.setPrj_name("OA系统");
        
        // 创建员工对象
        Developer dev_cj = new Developer();
        dev_cj.setD_name("曹吉");
        Developer dev_wc = new Developer();
        dev_wc.setD_name("王春");
        Developer dev_lz = new Developer();
        dev_lz.setD_name("老张");
        
        // 关系(1) 【项目方】
//      prj_ds.getDevelopers().add(dev_cj);
//      prj_ds.getDevelopers().add(dev_wc);// 电商系统（曹吉，王春）
//      
//      prj_oa.getDevelopers().add(dev_wc);
//      prj_oa.getDevelopers().add(dev_lz);// OA系统（王春，老张）
        
        //// 关系(2) 【员工方】
        dev_cj.getProjects().add(prj_ds);
        dev_wc.getProjects().add(prj_ds); 
        dev_wc.getProjects().add(prj_oa);
        dev_lz.getProjects().add(prj_oa);
        

        session.save(dev_cj);
        session.save(dev_wc);
        session.save(dev_lz);
        
        session.save(prj_ds);
        session.save(prj_oa);   // 如果不想执行保存员工操作,必须要设置级联保存
        
        session.getTransaction().commit();
        session.close();
    }
}
```

输出的数据库：

 t_developer 

| d_id | d_name |
| ---: | :----  |
| 1    | 曹吉   |
| 2    | 王春   |
| 3    | 老张   |

t_project

| prj_id | prj_name |
| ----:  | :----    |
| 1      | 电商系统 |
| 2      | OA系统   |


t_relation

| did  | prjId |
| ---: | ---:  |
| 1    | 1     |
| 2    | 1     |
| 2    | 2     |
| 3    | 2     |

#### 测试类【获取数据】

```java
//2 .设置inverse属性， 对获取数据影响？  无
public void get() {
    
    Session session = sf.openSession();
    session.beginTransaction();
    
    Project prj = (Project) session.get(Project.class, 1);
    System.out.println(prj.getPrj_name());
    System.out.println(prj.getDevelopers());
    
    
    session.getTransaction().commit();
    session.close();
}
```

#### 测试类【解除数据】

```java
//3. 设置inverse属性， 对解除关系影响？
// 有影响。
// inverse=false ,有控制权， 解除关系就是删除中间表的数据。
// inverse=true, 没有控制权，不能解除关系。
public void removeRelation() {
    
    Session session = sf.openSession();
    session.beginTransaction();
    
    Project prj = (Project) session.get(Project.class, 1);
    prj.getDevelopers().clear();
    
    session.getTransaction().commit();
    session.close();
}
```

#### 测试类【删除数据】

```java
//3. 设置inverse属性，对删除数据的影响?
// inverse=false, 有控制权。 先删除中间表数据，再删除自身。
// inverse=true, 没有控制权。 如果删除的数据有被引用，会报错！ 否则，才可以删除
public void deleteData() {
    
    Session session = sf.openSession();
    session.beginTransaction();
    
    Project prj = (Project) session.get(Project.class, 1);
    session.delete(prj);
    
    session.getTransaction().commit();
    session.close();
}
```
