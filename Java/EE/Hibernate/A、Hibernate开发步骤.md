[TOC]

# Hibernate框架

## ORM(对象关系映射)概念

> O,  Object  对象<br/>
> R， Realtion 关系  (关系型数据库: MySQL, Oracle…)<br/>
> M，Mapping  映射<br/>

### 搭建一个Hibernate环境，开发步骤

**1. 下载源码**

>版本：hibernate-distribution-3.6.0.Final

&emsp;hibernate3.jar核心  +  required 必须引入的(6个) +  jpa 目录  + 数据库驱动包

>hibernate3.jar<br/>
>antlr-2.7.6.jar<br/>
>commons-collections-3.1.jar<br/>
>dom4j-1.6.1.jar<br/>
>javassist-3.12.0.GA.jar<br/>
>jta-1.1.jar<br/>
>slf4j-api-1.6.1.jar<br/>
>mysql-connector-java-5.1.12-bin.jar<br/>

**3. 写对象以及对象的映射**<br/>

>Employee.java            对象<br/>
>Employee.hbm.xml        对象的映射 (映射文件)<br/>

**4. src/hibernate.cfg.xml**  主配置文件

>数据库连接配置
>加载所用的映射(*.hbm.xml)

**5. App.java  测试**

>Employee.java 对象

```java
import java.util.Date;
import javax.activation.DataSource;
public class Employee {

    private int empId;
    private String empName;
    private Date workDate;
}
```

>Employee.hbm.xml 对象的映射

```xml
<?xml version="1.0"?>
<!DOCTYPE hibernate-mapping PUBLIC 
    "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
    "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">

<hibernate-mapping package="com.theliang.a_hello"
    <class name="Employee" table="employee">
        <!-- 主键 ，映射-->
        <id name="empId" column="id">
            <generator class="native"/>
        </id>
        <!-- 非主键，映射 -->
        <property name="empName" column="empName"></property>
        <property name="workDate" column="workDate"></property>   
    </class>
</hibernate-mapping>
```

>hibernate.cfg.xml    主配置文件

```xml
<!DOCTYPE hibernate-configuration PUBLIC
    "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
    "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">

<hibernate-configuration>
    <session-factory>
        <!-- 数据库连接配置 -->
        <property name="hibernate.connection.driver_class">com.mysql.jdbc.Driver</property>
        <property name="hibernate.connection.url">jdbc:mysql:///hib_demo</property>
        <property name="hibernate.connection.username">root</property>
        <property name="hibernate.connection.password">root</property>
        <property name="hibernate.dialect">org.hibernate.dialect.MySQL5Dialect</property>
        <!-- 控制台输出sql语句-->
        <property name="hibernate.show_sql">true</property>
        <!-- 加载所有映射 -->
        <mapping resource="com/theliang/a_hello/Employee.hbm.xml"/>
    </session-factory>
</hibernate-configuration>
```

>App.java   测试类

```java
import java.util.Date;

import org.hibernate.SessionFactory;
import org.hibernate.Transaction;
import org.hibernate.cfg.Configuration;
import org.hibernate.classic.Session;

public class App {
    @Test
    public void testHello() throws Exception {
        // 对象
        Employee emp = new Employee();
        emp.setEmpName("班长");
        emp.setWorkDate(new Date());
        // 获取加载配置文件的管理类对象
        Configuration config = new Configuration();
        config.configure();  // 默认加载src/hibenrate.cfg.xml文件
        // 创建session的工厂对象
        SessionFactory sf = config.buildSessionFactory();
        // 创建session (代表一个会话，与数据库连接的会话)
        Session session = sf.openSession();
        // 开启事务
        Transaction tx = session.beginTransaction();
        //保存-数据库
        session.save(emp);
        // 提交事务
        tx.commit();
        // 关闭
        session.close();
        sf.close();
    }
}

```
