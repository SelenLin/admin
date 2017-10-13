[TOC]

# Hibernate.cfg.xml 主配置

`Hibernate.cfg.xml`：主配置文件中主要配置：数据库连接信息、其他参数、映射信息
>模板位置：`.\hibernate-distribution-3.6.0.Final\project\etc\Hibernate.cfg.xml`<br/>
>存放位置 :`./src/hibernate.cfg.xml`
<br/>

常用配置查看源码：`hibernate-distribution-3.6.0.Final\project\etc\hibernate.properties`

```xml
<!DOCTYPE hibernate-configuration PUBLIC
    "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
    "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <!-- 通常，一个session-factory节点代表一个数据库 -->
    <session-factory>
        <!-- 1. 数据库连接配置 -->
        <property name="hibernate.connection.driver_class">com.mysql.jdbc.Driver</property>
        <property name="hibernate.connection.url">jdbc:mysql:///hib_demo</property>
        <property name="hibernate.connection.username">root</property>
        <property name="hibernate.connection.password">root</property>
        <!-- 
            数据库方法配置， hibernate在运行的时候，会根据不同的方言生成符合当前数据库语法的sql
         -->
        <property name="hibernate.dialect">org.hibernate.dialect.MySQL5Dialect</property>
        <!-- 2. 其他相关配置 -->
        <!-- 2.1 显示hibernate在运行时候执行的sql语句 -->
        <property name="hibernate.show_sql">true</property>
        <!-- 2.2 格式化sql -->
        <property name="hibernate.format_sql">true</property>
        <!-- 2.3 自动建表 
            create:删除原表并创建新表
            update:如果没有表就创建,如果有则添加
        -->
        <property name="hibernate.hbm2ddl.auto">create</property> 
        
        <!-- 3. 加载所有映射 
            注:如果在java类中创建SessionFactory对象时,有addClass(T.class),则不必写下面的配置
        <mapping resource="com/theliang/hello/T.hbm.xml"/>
        -->
    </session-factory>
</hibernate-configuration>
```


## 数据库连接参数配置

例如：
    
> MySQL

    #hibernate.dialect org.hibernate.dialect.MySQLDialect
    #hibernate.dialect org.hibernate.dialect.MySQLInnoDBDialect
    #hibernate.dialect org.hibernate.dialect.MySQLMyISAMDialect
    #hibernate.connection.driver_class com.mysql.jdbc.Driver
    #hibernate.connection.url jdbc:mysql:///test
    #hibernate.connection.username gavin
    #hibernate.connection.password

### 自动建表

>Hibernate.properties

```properties
#hibernate.hbm2ddl.auto create-drop //每次在创建sessionFactory时候执行创建表；当调用sesisonFactory的close方法的时候，删除表！
#hibernate.hbm2ddl.auto create   //每次都重新建表； 如果表已经存在就先删除再创建
#hibernate.hbm2ddl.auto update  //如果表不存在就创建； 表存在就不创建；
#hibernate.hbm2ddl.auto validate  //(生成环境时候) 执行验证： 当映射文件的内容与数据库表结构不一样的时候就报错！
```
    
### 通过代码自动建表

```java
public class App_ddl {
    // 自动建表
    @Test
    public void createTable() throws Exception {
        // 创建配置管理类对象
        Configuration config = new Configuration();
        // 加载主配置文件
        config.configure(); 
        // 创建工具类对象
        SchemaExport export = new SchemaExport(config);
        // 建表
        // 第一个参数： 是否在控制台打印建表语句
        // 第二个参数： 是否执行脚本
        export.create(true, true);
    }
}
```

> 注：在使用代码自动建表的时候，`src/hivernate.cfg.xml`文件中需要加载对应的映射文件，同时可不使用自动建表的配置

