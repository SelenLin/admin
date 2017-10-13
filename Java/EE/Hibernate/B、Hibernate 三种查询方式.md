[TOC]

## Hibernate  Api

* *Configuration*&emsp;&emsp;配置管理类对象
    - `config.configure();`    加载主配置文件的方法(`hibernate.cfg.xml`) 默认加载`src/hibernate.cfg.xml`
    - `config.configure("cn/config/hibernate.cfg.xml");` 加载指定路径下指定名称的主配置文件
    - `config.buildSessionFactory();`&emsp;&emsp;创建session的工厂对象
* *SessionFactory*&emsp;&emsp;session的工厂（或者说代表了这个`hibernate.cfg.xml`配置文件）
    - `sf.openSession();`   创建一个sesison对象
    - `sf.getCurrentSession();`  创建session或取出session对象
* *Session*&emsp;&emsp;session对象维护了一个连接(Connection), 代表了与数据库连接的会话。Hibernate最重要的对象： 使用hibernate与数据库操作，都用到这个对象
    - `session.beginTransaction();` 开启一个事务； hibernate要求所有的与数据库的操作必须有事务的环境，否则报错！
    - `session.save(obj);`   保存一个对象
    - `session.delete(emp);`  删除一个对象
    - `session.update(emp);`  更新一个对象
    - `session.saveOrUpdate(emp);`  保存或者更新的方法：
        + 没有设置主键，执行保存；
        + 有设置主键，执行更新操作; 
        + 如果设置主键不存在则报错！
    - 主键查询
        + `session.get(Employee.class, 1);`    主键查询
        + `session.load(Employee.class, 1);`   主键查询 (支持懒加载)

## 三种查询方式

### HQL查询(Hibernate Query Language)
> HQL查询与SQL查询区别：<br/>

&emsp;`SQL`: (结构化查询语句)查询的是表以及字段;  不区分大小写。<br/>
&emsp;`HQL`: hibernate  query  language 即`hibernate`提供的面向对象的查询语言查询的是对象以及对象的属性。 区分大小写。<br/>

```java
    Session.save(Object);               //增加
    Session.delete(Object);             //删除
    Session.update(Object);             //修改
    Session.saveOrUpdate(Object);       //保存或修改(自动)
    List<T> list=Session.get(T.class,"键值"); //查询
    Query query=Session.createQuery("from TableName where '条件'");   //查询
```

### QBC查询/Criteria查询(Query By Criteria)

> 完全面向对象的查询

```java
    Criteria criteria=Session.createCriteria(T.class);
    //条件
    criteria.add(Restrictions.eq("键名","键值"));
    //执行
    List<T> list=criteria.list();
```

#### 拓展:判断表达式

| 名称   | 符号    | 意义                  |
| :----: | :-----: | :-----:               |
| 等于   | eq      | equal                 |
| 不等于 | nq      | not equal             |
| 大于   | gt      | greater than          |
| 小于   | lt      | less than             |
| 大等于 | ge      | greater than or equal |
| 小等于 | le      | less than or equal    |


### 本地SQL查询
> 复杂的查询，就要使用原生态的sql查询，也可以，就是本地sql查询的支持！
    (缺点： 不能跨数据库平台！)

```java
    // 把每一行记录封装为对象数组，再添加到list集合
    SQLQuery sqlQuery1=Session.createSQLQuery("select * from TableName");
    sqlQuery1.list();//输出的是地址

    // 把每一行记录封装为 指定的对象类型
    SQLQuery sqlQuery2 = session.createSQLQuery("select * from employee").addEntity(Employee.class);
    sqlQuery2.list();//输出的是对象数据 
```

### 共性问题
* `ClassNotFoundException…`：缺少jar文件！
* 如果程序执行程序，hibernate也有生成sql语句，但数据没有结果影响：问题一般是事务忘记提交……


## Hibernate crud

>HibernateUtils.java

```java
import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;
import org.hibernate.classic.Session;

public class HibernateUtils {
    private static SessionFactory sf;
    static{
       // sf=new Configuration().configure().buildSessionFactory();
   //或者
        // 获取加载配置文件的管理类对象
        Configuration config=new Configuration();
        // 默认加载src/hibenrate.cfg.xml文件
        config.configure();
        // 创建session的工厂对象
        sf=config.buildSessionFactory();
    }
    public static Session getSession(){
        return sf.openSession();
    }
}
```

### 增加

```java
    public void save(Employee emp) {
        Session session = null;
        Transaction tx = null;
        try {
            session = HibernateUtils.getSession();
            tx = session.beginTransaction();
            // 执行保存操作
            session.save(emp);
        } catch (Exception e) {
            throw new RuntimeException(e);
        } finally {
            tx.commit();
            session.close();
        }
    }
```


### 删除

>在Hibernate框架查询时，用Serializable代替Int,String,Long....
>
>因为Int，String，Long都实现了Serializable这个接口，用Serializable代替，可以实现代码的通用性。

```java
    public void delete(Serializable id) {
        Session session = null;
        Transaction tx = null;
        try {
            session = HibernateUtils.getSession();
            tx = session.beginTransaction();
            // 先根据id查询对象，再判断删除
            Object obj = session.get(Employee.class, id);
            if (obj != null) {
                session.delete(obj);
            }
        } catch (Exception e) {
            throw new RuntimeException(e);
        } finally {
            tx.commit();
            session.close();
        }
    }
}
```

### 修改

```java
    public void update(Employee emp) {
        Session session = null;
        Transaction tx = null;
        try {
            session = HibernateUtils.getSession();
            tx = session.beginTransaction();
            //更新操作
            session.update(emp);
        } catch (Exception e) {
            throw new RuntimeException(e);
        } finally {
            tx.commit();
            session.close();
        }
    }
```

### 查询

#### 根据主键查询

```java
  public Employee findById(Serializable id) {
        Session session = null;
        Transaction tx = null;
        try {
            // 获取Session
            session = HibernateUtils.getSession();
            // 开启事务
            tx = session.beginTransaction();
            // 主键查询
            return (Employee) session.get(Employee.class, id);
        } catch (Exception e) {
            throw new RuntimeException(e);
        } finally {
            tx.commit();
            session.close();
        }
    }
```

#### 根据名称查询

```java
    public List<Employee> getAll(String employeeName) {
        Session session = null;
        Transaction tx = null;
        try {
            session = HibernateUtils.getSession();
            tx = session.beginTransaction();
            Query q =session.createQuery("from Employee where empName=?");
            //对应查询语句中的‘？’ 注意：参数索引从0开始
            q.setParameter(0, employeeName);
            // 执行查询
            return q.list();
        } catch (Exception e) {
            throw new RuntimeException(e);
        } finally {
            tx.commit();
            session.close();
        }
    }
```

#### 查询全部

```java
    public List<Employee> getAll() {
        Session session = null;
        Transaction tx = null;
        try {
            session = HibernateUtils.getSession();
            tx = session.beginTransaction();
            // HQL查询
            Query q = session.createQuery("from Employee");
            return q.list();
        } catch (Exception e) {
            throw new RuntimeException(e);
        } finally {
            tx.commit();
            session.close();
        }
    }
```

#### 分布查询

```java
    public List<Employee> getAll(int index, int count) {
        Session session = null;
        Transaction tx = null;
        try {
            session = HibernateUtils.getSession();
            tx = session.beginTransaction();
            Query q = session.createQuery("from Employee");
            // 设置分页参数
            q.setFirstResult(index);  // 查询的其实行 
            q.setMaxResults(count);   // 查询返回的行数
            
            List<Employee> list = q.list();
            return list;
        } catch (Exception e) {
            throw new RuntimeException(e);
        } finally {
            tx.commit();
            session.close();
        }
    }
```
