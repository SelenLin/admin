[TOC]

# 一、对象的状态

> Hibernate中对象的状态： 临时/瞬时状态、持久化状态、游离状态。

## 临时状态
> 特点： 
>> 直接new出来的对象;  
>> 不处于session的管理;  
>> 数据库中没有对象的记录;  

## 持久化状态
当调用session的`save`/`saveOrUpdate`/`get`/`load`/`list`等方法的时候，对象就是持久化状态。处于持久化状态的对象，当对对象属性进行更改的时候，会反映到数据库中!

> 特点：
>> 处于session的管理;  
>> 数据库中有对应的记录;  

## 游离状态

> 特点
>> 不处于session的管理；  
>> 数据库中有对应的记录；  
>> Session关闭后，对象的状态；  

# 二、一级缓存

> 为什么要用缓存？
    目的：减少对数据库的访问次数！从而提升hibernate的执行效率！

## 概念

1. Hibenate中一级缓存，也叫做session的缓存，**它可以在session范围内减少数据库的访问次数！**  只在session范围有效！ Session关闭，一级缓存失效！  
2. 当调用session的save/saveOrUpdate/get/load/list/iterator方法的时候，都会把对象放入session的缓存中。  
3. Session的缓存由hibernate维护， 用户不能操作缓存内容； 如果想操作缓存内容，必须通过hibernate提供的`evict`/`clear`方法操作。

特点：  
&emsp;只在(当前)session范围有效，作用时间短，效果不是特别明显！
    在短时间内多次操作数据库，效果比较明显！

## API

缓存相关几个方法的作用

```java
session.flush();       //让一级缓存与数据库同步
session.evict(arg0);   //清空一级缓存中指定的对象
session.clear();       //清空一级缓存中缓存的所有对象
```

在什么情况用上面方法？  
> 批量操作使用方法：

```java
Session.flush();   // 先与数据库同步
Session.clear();   // 再清空一级缓存内容
```

### 测试一级缓存

```java
public void testCache()throws Exception{
        Session session=sf.openSession();
        session.beginTransaction();
        
        User user=null;
        
        user=(User) session.get(User.class, 1);
        System.out.println(user);
        
        user=(User) session.get(User.class, 1);
        System.out.println(user);
           
        session.getTransaction().commit();
        session.close();
}
/**控制台输出：
 *      Hibernate: select user0_.id as id0_0_, user0_.name as name0_0_ from t_User user0_ where user0_.id=?
 *      User [userId=1, userName=123]
 *      User [userId=1, userName=123]
 */
```

### 刷新缓存

```java
public void flush() throws Exception {
        Session session = sf.openSession();
        session.beginTransaction();
        
        User user = null;
        user = (User) session.get(User.class, 1);
        
        user.setUserName("Jack");
        // 缓存数据与数据库同步
        session.flush();
        
        user.setUserName("Jack_new");
        
        session.getTransaction().commit();  // 相当于执行了session.flush();
        session.close();
}
/**控制台输出：
 * 刷新两次则输出两条update
 *      Hibernate: select user0_.id as id0_0_, user0_.name as name0_0_ from t_User user0_ where user0_.id=?
 *      Hibernate: update t_User set name=? where id=?
 *      Hibernate: update t_User set name=? where id=?  
 */
```

### 清除缓存

```java
public void clear()throws Exception{
        Session session=sf.openSession();
        session.beginTransaction();
        
        User user=null;
        //查询
        user=(User) session.get(User.class, 1);
        
        //清空缓存内容
        session.clear();//清空所有
//      session.evict(user);//清除指定
        System.out.println(user);
        
        user=(User) session.get(User.class, 1);
        System.out.println(user);
        
        session.getTransaction().commit();
        session.close();
}
/**控制台输出：
 *      Hibernate: select user0_.id as id0_0_, user0_.name as name0_0_ from t_User user0_ where user0_.id=?
 *      User [userId=1, userName=123]
 *      Hibernate: select user0_.id as id0_0_, user0_.name as name0_0_ from t_User user0_ where user0_.id=?
 *      User [userId=1, userName=123]
 */
```

### 问题：不同的session是否会共享缓存数据?

> 不会。  
> User1  u1 = Session1.get(User.class,1);   把u1对象放入session1的缓存  
> Session2.update(u1);     把u1放入session2的缓存  
> 
> U1.setName(‘new Name’);  
> 
> 如果生成2条update sql， 说明不同的session使用不同的缓存区，不能共享。

```java
public void sessionTest() throws Exception {
        Session session1 = sf.openSession();
        session1.beginTransaction();
        Session session2 = sf.openSession();
        session2.beginTransaction();
        
        // user放入session1的缓存区
        User user = (User) session1.get(User.class, 1);
        // user放入session2的缓存区
        session2.update(user);
        
        // 修改对象
        user.setUserName("New Name");  // 2条update
/**
 *      Hibernate: select user0_.id as id0_0_, user0_.name as name0_0_ from t_User user0_ where user0_.id=?
 *      Hibernate: update t_User set name=? where id=?
 *      Hibernate: update t_User set name=? where id=?
 */
        session1.getTransaction().commit();  // session1.flush();
        session1.close();
        session2.getTransaction().commit();  // session2.flush();
        session2.close();
}
```

### 问题：list与iterator查询的区别？

> 1. 返回的类型不一样，list()返回List,iterate()返回Iterator  
> 
> 2. 获取数据的方式不一样，list()会直接查数据库,iterate()会先到数据库中把id都取出来，然后真正要遍历某个对象的时候先到缓存中找,如果找不到,以id为条件再发一条sql到数据库,这样如果缓存中没有数据,则查询数据库的次数为n+1。
> 
> 3. iterate会查询二级缓存，　list只会查询一级缓存。  
> 
> 4. list()中返回的List中每个对象都是原本的对象,iterate()中返回的对象是代理对象.(debug可以发现)
> 
> 5. list()查询的记录会放入缓存中，但不会从缓存中获取数据。iterator()会放入缓存中，也会从缓存中获取数据！
> 
> 6.    * list()方法在执行时，直接运行查询结果所需要的查询语句。
>       * iterator()方法则是先执行得到对象ID的查询，然后在根据每个ID值去取得所要查询的对象。
>       * 因此：对于list()方式的查询通常只会执行一个SQL语句，而对于iterator()方法的查询则可能需要执行`N+1`条SQL语句(N为结果集中的记录数).
> 
> 7. 结果集的处理方法不同:
>       * list()方法会一次取出所有的结果集对象，而且他会依据查询的结果初始化所有的结果集对象。如果在结果集非常庞大的时候会占据非常多的内存，甚至会造成内存溢出的情况发生。
>       * iterator()方法在执行时不会一次初始化所有的对象，而是根据对结果集的访问情况来初始化对象。一次在访问中可以控制缓存中对象的数量，以避免占用过多的缓存，导致内存溢出情况的发生。


#### list 方法

```java
public void list(){
        Session session=sf.openSession();
        session.beginTransaction();
        // HQL查询
        Query q=session.createQuery("from User");
        // list()方法
        List<User> list=q.list();
        
        for (int i = 0; i < list.size(); i++) {
            System.out.println(list.get(i));
        }
        session.getTransaction().commit();
        session.close();
}
/**控制台输出：
*   Hibernate: select user0_.id as id0_, user0_.name as name0_ from t_User user0_
*   User [userId=1, userName=New Name]
*   User [userId=2, userName=123]
*   User [userId=3, userName=张三123]
*/
```

#### iterator 方法

```java
public void iter(){
        Session session=sf.openSession();
        session.beginTransaction();
        // HQL查询
        Query q=session.createQuery("from User");
        // iterator()方法
        Iterator<User> it = q.iterate();
        while (it.hasNext()) {
            // 得到当前迭代的每一个对象
            User user=it.next();
            System.out.println(user);
        }
        session.getTransaction().commit();
        session.close();
}
/**控制台输出：先从数据库取出ID，然后迭代的时候再根据ID查找数据
*    Hibernate: select user0_.id as col_0_0_ from t_User user0_
*    Hibernate: select user0_.id as id0_0_, user0_.name as name0_0_ from t_User user0_ where user0_.id=?
*    User [userId=1, userName=New Name]
*    Hibernate: select user0_.id as id0_0_, user0_.name as name0_0_ from t_User user0_ where user0_.id=?
*    User [userId=2, userName=123]
*    Hibernate: select user0_.id as id0_0_, user0_.name as name0_0_ from t_User user0_ where user0_.id=?
*    User [userId=3, userName=张三123]
*/
```

#### 缓存list与iterator

```java
public void cache() throws Exception {
        Session session = sf.openSession();
        session.beginTransaction();
        
        /**************执行两次list*****************/
        Query q = session.createQuery("from User");
        List<User> list = q.list();      // 【会放入】
        for (int i=0; i<list.size(); i++){
            System.out.println(list.get(i));
        }
        System.out.println("=========第二次list===========");
        list = q.list();                // 【会放入,不取出】
        for (int i=0; i<list.size(); i++){
            System.out.println(list.get(i));
        }
/**
*   Hibernate: select user0_.id as id0_, user0_.name as name0_ from t_User user0_
*   User [userId=1, userName=New Name]
*   User [userId=2, userName=123]
*   User [userId=3, userName=张三123]
*   =========list===========
*   Hibernate: select user0_.id as id0_, user0_.name as name0_ from t_User user0_
*   User [userId=1, userName=New Name]
*   User [userId=2, userName=123]
*   User [userId=3, userName=张三123]
 */
        /**************执行两次iteator******************/
        Query q = session.createQuery("from User ");
        Iterator<User> it = q.iterate();        // 【放入缓存】
        while(it.hasNext()){
            User user = it.next();
            System.out.println(user);
        }
        System.out.println("==========第二次iterate===========");
        it = q.iterate();                       // 【也会从缓存中取】
        while(it.hasNext()){
            User user = it.next();
            System.out.println(user);
        }
        
        session.getTransaction().commit();  
        session.close();
/**
*   Hibernate: select user0_.id as col_0_0_ from t_User user0_
*   Hibernate: select user0_.id as id0_0_, user0_.name as name0_0_ from t_User user0_ where user0_.id=?
*   User [userId=1, userName=New Name]
*   Hibernate: select user0_.id as id0_0_, user0_.name as name0_0_ from t_User user0_ where user0_.id=?
*   User [userId=2, userName=123]
*   Hibernate: select user0_.id as id0_0_, user0_.name as name0_0_ from t_User user0_ where user0_.id=?
*   User [userId=3, userName=张三123]
*   ==========iterate===========
*   Hibernate: select user0_.id as col_0_0_ from t_User user0_
*   User [userId=1, userName=New Name]
*   User [userId=2, userName=123]
*   User [userId=3, userName=张三123]
 */
}
```

#### list放入缓存，iterator取出

```java
public void list_iterator() throws Exception {
        Session session = sf.openSession();
        session.beginTransaction();
        
        // 得到Query接口的引用
        Query q = session.createQuery("from User ");
        
        // 先list  【会放入缓存，但不会从缓存中获取数据】
        List<User> list = q.list(); 
        for (int i=0; i<list.size(); i++){
            System.out.println(list.get(i));
        }
        
        // 再iteraotr  (会从缓存中取)
        Iterator<User> it = q.iterate();
        while(it.hasNext()){
            User user = it.next();
            System.out.println(user);
        }
        
        session.getTransaction().commit();  
        session.close();
/**
*   Hibernate: select user0_.id as id0_, user0_.name as name0_ from t_User user0_
*   User [userId=1, userName=New Name]
*   User [userId=2, userName=123]
*   User [userId=3, userName=张三123]
*   Hibernate: select user0_.id as col_0_0_ from t_User user0_
*   User [userId=1, userName=New Name]
*   User [userId=2, userName=123]
*   User [userId=3, userName=张三123]
*/
}
```
