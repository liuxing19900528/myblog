Hibernate 学习 之 二级缓存


马哥的淘宝店铺 https://shop592330910.taobao.com/

##session级别的缓存
1.首先我们测试，获取100号员工的名称，这个时候会发送几条sql语句呢？
```java


    @Test
    public void testHibernateCache() { //马哥的淘宝店铺 https://shop592330910.taobao.com/
        //hibernate 二级缓存
        Employee employee = (Employee) session.get(Employee.class, 100);

        System.out.println(employee.getName());


    }

```
```text
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
King

```
我们可以看到是发送了一条sql语句。

2.接下来我们再改。
```java

    @Test
    public void testHibernateCache() {//马哥的淘宝店铺 https://shop592330910.taobao.com/
        //hibernate 二级缓存
        Employee employee = (Employee) session.get(Employee.class, 100);
        System.out.println(employee.getName());

        Employee employee1 = (Employee) session.get(Employee.class, 100);
        System.out.println(employee1.getName());


    }

```
这次我们是获取了100号员工名称，打印，然后接着又获取了100号员工名字，打印，这个时候会打印几条sql语句呢？
```sql
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
King
King


```
我们可以看到只发送了一条sql语句。为什么呢？
因为session有个缓存，第一次获取到时候会把这个员工信息放到缓存中。  
第二次在获取就不会再去查询数据库。

马哥的淘宝店铺 https://shop592330910.taobao.com/


我们这个测试代码中 是在 ``` public void init() {  ``` 方法中初始化的session，  
然后是在```  public void destroy() {  ```   方法中关闭的session。  
我们的测试方法是在他们之间执行的。

3.最后  
这个时候我们在两次获取100号员工信息之间 把session关闭，
然后在重新打开一个新的session，这个时候又会是上面情况呢？

```java

    @Test
    public void testHibernateCache1() {//马哥的淘宝店铺 https://shop592330910.taobao.com/
        //hibernate 二级缓存
        Employee employee = (Employee) session.get(Employee.class, 100);
        System.out.println(employee.getName());
               //5.提交事务
        transaction.commit();

        //6.关闭session对象
        session.close(); 
        
        //再次重新打开一个session
        session = sessionFactory.openSession();
        transaction = session.beginTransaction();        

        Employee employee1 = (Employee) session.get(Employee.class, 100);
        System.out.println(employee1.getName());


    }

```

```text

Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
King
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
King
=destroy=

```
通过上面输出发现最后是发送了2次sql语句。session关了缓存就没了。马哥的淘宝店铺 https://shop592330910.taobao.com/

接下来我们说二级缓存。  
二级缓存做的就是 上面把session关了，还能只发送1条sql语句。  马哥的淘宝店铺 https://shop592330910.taobao.com/


hibernate中有个2个级别的缓存  
* 第一级别是 session级别缓存  
* 第二级别是 sessionFactory缓存，是整个进程级别的。

马哥的淘宝店铺 https://shop592330910.taobao.com/

   * 内置缓存
   * 外置缓存，一个可配置的缓存插件

适合放入二级缓存的数据：马哥的淘宝店铺 https://shop592330910.taobao.com/

   * 1.很少被修改。
   * 2.不是很重要的数据，允许出现偶尔的并发问题。

不适合放入二级缓存的数据 马哥的淘宝店铺 https://shop592330910.taobao.com/

   * 1.经常被修改。
   * 2.财务数据，绝对不允许出现并发问题。
   * 3.与其他应用共享的数据。
    
马哥的淘宝店铺 https://shop592330910.taobao.com/


##二级缓存的并发访问策略

两个并发事务同时访问持久层的缓存的相同的数据时，也有可能出现各类并发问题。  
二级缓存可以设定以下四种并发访问策略，每一种策略对应一种事务隔离级别。
   
>马哥的淘宝店铺 https://shop592330910.taobao.com/
>
>   1.非严格读写（nonstrict-read-write）:不保证缓存与数据库中的数据的一致性，提供
>   read uncommmit 事务隔离级别，对于极少被修改，而且允许脏读的数据可以采用这种策略。
>
>   马哥的淘宝店铺 https://shop592330910.taobao.com/ 
>
>
>   2.读写型（read-write）提供read commited数据隔离级别，对于经常读，但是很少被修改的数据，可以采用这种隔离类型，因为它可以放在脏读。
>
>   马哥的淘宝店铺 https://shop592330910.taobao.com/
>   3.事务型（transactional），仅在受关联环境下适用，它提供了repeatable read事务隔离级别，
>
>   马哥的淘宝店铺 https://shop592330910.taobao.com/
>
>   4.只读型（read only）提供serializabale数据隔离级别，对于从来不会被修改的数据，可以采用这种方式


二级缓存可以使用的插件
>  * 1.ehcache   
>  * 2.oscache



##下面介绍EHCache的使用

1.首先要加入ehcache的jar包   马哥的淘宝店铺 https://shop592330910.taobao.com/
```xml

        <dependency>马哥的淘宝店铺 https://shop592330910.taobao.com/
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-ehcache</artifactId>
            <version>4.1.1.Final</version>
        </dependency>

```
这个ehcache的依赖很多，这里选择使用hibernate-ehcache这个。 
这个要特别注意，目前马哥还没弄明白这几个jar包的区别。

2.配置hibernate.cfg.xml 配置文件   马哥的淘宝店铺 https://shop592330910.taobao.com/
```xml

        <!--配置使用二级缓存-->
        <property name="cache.use_second_level_cache">true</property>
        <!--配置使用的二级缓存的产品 -->
        <property name="hibernate.cache.region.factory_class">org.hibernate.cache.ehcache.EhCacheRegionFactory</property>
        
        <!--这个也可以配置到hbm.xml 中配置 注意这些节点有顺序的-->
        <class-cache class="com.mamh.hibernate.hql.entities.Employee" usage="read-write"/><!-- 配置对哪些类使用二级缓存 -->

```




3.然后在执行我们的测试代码  马哥的淘宝店铺 https://shop592330910.taobao.com/
```java


    @Test
    public void testHibernateCache1() {//马哥的淘宝店铺 https://shop592330910.taobao.com/
        //hibernate 二级缓存
        Employee employee = (Employee) session.get(Employee.class, 100);
        System.out.println(employee.getName());
               //5.提交事务
        transaction.commit();

        //6.关闭session对象
        session.close();

        //再次重新打开一个session
        session = sessionFactory.openSession();
        transaction = session.beginTransaction();

        Employee employee1 = (Employee) session.get(Employee.class, 100);
        System.out.println(employee1.getName());


    }


```
```text

Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
King
King
=destroy=

```
通过结果我们明显的发现值发送了一条sql语句。马哥的淘宝店铺 https://shop592330910.taobao.com/

开可以通过注释class-cache 这个来观察二级缓存的打开关闭情况。



4.我们看集合的二级缓存
```java

    @Test
    public void testHibernateCache2() {//马哥的淘宝店铺 https://shop592330910.taobao.com/
        Department department = (Department) session.get(Department.class, 80);
        System.out.println(department.getName());
        System.out.println(department.getEmps().size());
    }

```
```text
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        department0_.id as id0_0_,
        department0_.name as name0_0_ 
    from
        hb_department department0_ 
    where
        department0_.id=?
Sales
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        emps0_.dept_id as dept5_0_1_,
        emps0_.id as id1_,
        emps0_.id as id1_0_,
        emps0_.name as name1_0_,
        emps0_.salary as salary1_0_,
        emps0_.email as email1_0_,
        emps0_.dept_id as dept5_1_0_ 
    from
        hb_employee emps0_ 
    where
        emps0_.dept_id=?
34


```    
我们看到这里employee采用了延迟加载，我们代码是连续的打印

这里我们获取两次，打印两次。
```java

    @Test
    public void testHibernateCache2() {//马哥的淘宝店铺 https://shop592330910.taobao.com/
        Department department = (Department) session.get(Department.class, 80);
        System.out.println(department.getName());
        System.out.println(department.getEmps().size());



        Department department1 = (Department) session.get(Department.class, 80);
        System.out.println(department1.getName());
        System.out.println(department1.getEmps().size());
    }

```
```

Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        department0_.id as id0_0_,
        department0_.name as name0_0_ 
    from
        hb_department department0_ 
    where
        department0_.id=?
Sales
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        emps0_.dept_id as dept5_0_1_,
        emps0_.id as id1_,
        emps0_.id as id1_0_,
        emps0_.name as name1_0_,
        emps0_.salary as salary1_0_,
        emps0_.email as email1_0_,
        emps0_.dept_id as dept5_1_0_ 
    from
        hb_employee emps0_ 
    where
        emps0_.dept_id=?
34
Sales
34

```
我们可以看到sql还是2条，这个是有session缓存导致的  马哥的淘宝店铺 https://shop592330910.taobao.com/


下面我们如法炮制，关闭一个session，然后再打开session。
```java

    @Test
    public void testHibernateCache2() {//马哥的淘宝店铺 https://shop592330910.taobao.com/
        Department department1 = (Department) session.get(Department.class, 80);
        System.out.println("deapatment1 = " + department1.getName());
        System.out.println("deapatment1 = " + department1.getEmps().size());

        //5.提交事务  马哥的淘宝店铺 https://shop592330910.taobao.com/
        transaction.commit();

        //6.关闭session对象
        session.close();

        //再次重新打开一个session 马哥的淘宝店铺 https://shop592330910.taobao.com/
        session = sessionFactory.openSession();
        transaction = session.beginTransaction();  

        Department department2 = (Department) session.get(Department.class, 80);
        System.out.println("deapatment2 = " + department2.getName());
        System.out.println("deapatment2 = " + department2.getEmps().size());
    }

```
```text
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        department0_.id as id0_0_,
        department0_.name as name0_0_ 
    from
        hb_department department0_ 
    where
        department0_.id=?
deapatment1 = Sales
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        emps0_.dept_id as dept5_0_1_,
        emps0_.id as id1_,
        emps0_.id as id1_0_,
        emps0_.name as name1_0_,
        emps0_.salary as salary1_0_,
        emps0_.email as email1_0_,
        emps0_.dept_id as dept5_1_0_ 
    from
        hb_employee emps0_ 
    where
        emps0_.dept_id=?
deapatment1 = 34
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        department0_.id as id0_0_,
        department0_.name as name0_0_ 
    from
        hb_department department0_ 
    where
        department0_.id=?
deapatment2 = Sales
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        emps0_.dept_id as dept5_0_1_,
        emps0_.id as id1_,
        emps0_.id as id1_0_,
        emps0_.name as name1_0_,
        emps0_.salary as salary1_0_,
        emps0_.email as email1_0_,
        emps0_.dept_id as dept5_1_0_ 
    from
        hb_employee emps0_ 
    where
        emps0_.dept_id=?
deapatment2 = 34


```
这个我们发现是发送了4条sql语句。我们没有在department，和其中的集合设置二级缓存。

下面我们对department设置二级缓存。
```xml
<!--<class-cache class="com.mamh.hibernate.hql.entities.Employee" usage="read-write"/>-->

<!--<class-cache class="com.mamh.hibernate.hql.entities.Department" usage="read-write"/>-->
<!--<collection-cache collection="com.mamh.hibernate.hql.entities.Department.emps" usage="read-write"/>-->

注意这里我们只打开了这一个department的二级缓存配置  马哥的淘宝店铺 https://shop592330910.taobao.com/
<class-cache class="com.mamh.hibernate.hql.entities.Department" usage="read-write"/>

```

```text
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        department0_.id as id0_0_,
        department0_.name as name0_0_ 
    from
        hb_department department0_ 
    where
        department0_.id=?
deapatment1 = Sales
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        emps0_.dept_id as dept5_0_1_,
        emps0_.id as id1_,
        emps0_.id as id1_0_,
        emps0_.name as name1_0_,
        emps0_.salary as salary1_0_,
        emps0_.email as email1_0_,
        emps0_.dept_id as dept5_1_0_ 
    from
        hb_employee emps0_ 
    where
        emps0_.dept_id=?
deapatment1 = 34
deapatment2 = Sales
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        emps0_.dept_id as dept5_0_1_,
        emps0_.id as id1_,
        emps0_.id as id1_0_,
        emps0_.name as name1_0_,
        emps0_.salary as salary1_0_,
        emps0_.email as email1_0_,
        emps0_.dept_id as dept5_1_0_ 
    from
        hb_employee emps0_ 
    where
        emps0_.dept_id=?
deapatment2 = 34

```
设置了department的二级缓存我们发现sql语句少了1条，但是employee的还是2条。

马哥的淘宝店铺 https://shop592330910.taobao.com/

接下来我们设置集合employee的二级缓存

```xml
<!--<class-cache class="com.mamh.hibernate.hql.entities.Employee" usage="read-write"/>-->
注意这里我们还没有打开employee的二级缓存，特别注意！！！
<class-cache class="com.mamh.hibernate.hql.entities.Department" usage="read-write"/>
<collection-cache collection="com.mamh.hibernate.hql.entities.Department.emps" usage="read-write"/>

```
```text
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        department0_.id as id0_0_,
        department0_.name as name0_0_ 
    from
        hb_department department0_ 
    where
        department0_.id=?
deapatment1 = Sales
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        emps0_.dept_id as dept5_0_1_,
        emps0_.id as id1_,
        emps0_.id as id1_0_,
        emps0_.name as name1_0_,
        emps0_.salary as salary1_0_,
        emps0_.email as email1_0_,
        emps0_.dept_id as dept5_1_0_ 
    from
        hb_employee emps0_ 
    where
        emps0_.dept_id=?
deapatment1 = 34
deapatment2 = Sales
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
Hibernate: 
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
Hibernate: 
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
Hibernate: 
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
Hibernate: 
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
Hibernate: 
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
Hibernate: 
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
Hibernate:马哥的淘宝店铺 https://shop592330910.taobao.com/ 
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
Hibernate:马哥的淘宝店铺 https://shop592330910.taobao.com/ 
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
Hibernate:马哥的淘宝店铺 https://shop592330910.taobao.com/ 
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        employee0_.id as id1_0_,
        employee0_.name as name1_0_,
        employee0_.salary as salary1_0_,
        employee0_.email as email1_0_,
        employee0_.dept_id as dept5_1_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.id=?
deapatment2 = 34


```
这个时候我们发现employee的sql语句反而变多了。。。马哥的淘宝店铺 https://shop592330910.taobao.com/
  
这个是因为没有对employee使用二级缓存，我们只对集合使用了二级缓存，  马哥的淘宝店铺 https://shop592330910.taobao.com/  
这个只缓存了employee的ID，我们通过那些sql  马哥的淘宝店铺 https://shop592330910.taobao.com/  
也能看出来是一个一个通过ID来查询出来employee的。马哥的淘宝店铺 https://shop592330910.taobao.com/  


最后我们给employee也设置二级缓存
```xml

<class-cache class="com.mamh.hibernate.hql.entities.Employee" usage="read-write"/>

<class-cache class="com.mamh.hibernate.hql.entities.Department" usage="read-write"/>
<collection-cache collection="com.mamh.hibernate.hql.entities.Department.emps" usage="read-write"/>

```
```text

Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        department0_.id as id0_0_,
        department0_.name as name0_0_ 
    from
        hb_department department0_ 
    where
        department0_.id=?
deapatment1 = Sales
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        emps0_.dept_id as dept5_0_1_,
        emps0_.id as id1_,
        emps0_.id as id1_0_,
        emps0_.name as name1_0_,
        emps0_.salary as salary1_0_,
        emps0_.email as email1_0_,
        emps0_.dept_id as dept5_1_0_ 
    from
        hb_employee emps0_ 
    where
        emps0_.dept_id=?
deapatment1 = 34
deapatment2 = Sales
deapatment2 = 34

```
这个输出已经是我们期望的，只发送了2条sql语句，这个就是对集合设置了二级缓存。马哥的淘宝店铺 https://shop592330910.taobao.com/    
马哥的淘宝店铺 https://shop592330910.taobao.com/

集合的二级缓存也可以配置到hbm文件中，配置到set元素下面。

## ehcache.xml 配置文件介绍

这个配置文件可以从hibernate-release-4.3.5.Final.zip 包中得到。
https://sourceforge.net/projects/hibernate/files/hibernate4/4.3.5.Final/hibernate-release-4.3.5.Final.zip/download


马哥的淘宝店铺 https://shop592330910.taobao.com/
```xml

<ehcache>

    <!-- Sets the path to the directory where cache .data files are created.

         If the path is a Java System Property it is replaced by
         its value in the running VM.

         The following properties are translated:马哥的淘宝店铺 https://shop592330910.taobao.com/
         user.home - User's home directory
         user.dir - User's current working directory
         java.io.tmpdir - Default temp file path -->
    <diskStore path="java.io.tmpdir"/>


    <!--Default Cache configuration. These will applied to caches programmatically created through
        the CacheManager.

        The following attributes are required for defaultCache:马哥的淘宝店铺 https://shop592330910.taobao.com/

        maxInMemory       - Sets the maximum number of objects that will be created in memory
        eternal           - Sets whether elements are eternal. If eternal,  timeouts are ignored and the element
                            is never expired.
        timeToIdleSeconds - Sets the time to idle for an element before it expires. Is only used
                            if the element is not eternal. Idle time is now - last accessed time
        timeToLiveSeconds - Sets the time to live for an element before it expires. Is only used
                            if the element is not eternal. TTL is now - creation time
        overflowToDisk    - Sets whether elements can overflow to disk when the in-memory cache
                            has reached the maxInMemory limit.

        -->
    <defaultCache
        maxElementsInMemory="10000"
        eternal="false"
        timeToIdleSeconds="120"
        timeToLiveSeconds="120"
        overflowToDisk="true"
        />

    <!--Predefined caches.  Add your cache configuration settings here.
        If you do not have a configuration for your cache a WARNING will be issued when the
        CacheManager starts

        The following attributes are required for defaultCache:马哥的淘宝店铺 https://shop592330910.taobao.com/

        name              - Sets the name of the cache. This is used to identify the cache. It must be unique.
        maxInMemory       - Sets the maximum number of objects that will be created in memory
        eternal           - Sets whether elements are eternal. If eternal,  timeouts are ignored and the element
                            is never expired.
        timeToIdleSeconds - Sets the time to idle for an element before it expires. Is only used
                            if the element is not eternal. Idle time is now - last accessed time
        timeToLiveSeconds - Sets the time to live for an element before it expires. Is only used
                            if the element is not eternal. TTL is now - creation time
        overflowToDisk    - Sets whether elements can overflow to disk when the in-memory cache
                            has reached the maxInMemory limit.

        -->

    <!-- Sample cache named sampleCache1马哥的淘宝店铺 https://shop592330910.taobao.com/
        This cache contains a maximum in memory of 10000 elements, and will expire
        an element if it is idle for more than 5 minutes and lives for more than
        10 minutes.

        If there are more than 10000 elements it will overflow to the
        disk cache, which in this configuration will go to wherever java.io.tmp is
        defined on your system. On a standard Linux system this will be /tmp"
        -->
    <cache name="sampleCache1"
        maxElementsInMemory="10000"
        eternal="false"
        timeToIdleSeconds="300"
        timeToLiveSeconds="600"
        overflowToDisk="true"
        />

    <!-- Sample cache named sampleCache2马哥的淘宝店铺 https://shop592330910.taobao.com/
        This cache contains 1000 elements. Elements will always be held in memory.
        They are not expired. -->
    <cache name="sampleCache2"
        maxElementsInMemory="1000"
        eternal="true"
        timeToIdleSeconds="0"
        timeToLiveSeconds="0"
        overflowToDisk="false"
        /> -->

    <!-- Place configuration for your caches following 马哥的淘宝店铺 https://shop592330910.taobao.com/-->

</ehcache>


```
马哥的淘宝店铺 https://shop592330910.taobao.com/

* 磁盘存储路径
```xml
    <diskStore path="java.io.tmpdir"/>
```
马哥的淘宝店铺 https://shop592330910.taobao.com/


* 默认的配置
```xml

<defaultCache
        maxElementsInMemory="10000"
        eternal="false"
        timeToIdleSeconds="120"
        timeToLiveSeconds="120"
        overflowToDisk="true"
        />

```
马哥的淘宝店铺 https://shop592330910.taobao.com/
* 命名的缓存区域设置

对类而言 命名是 类的全类名
对类里面的集合而言 是 全类名 加上 集合 名称
```xml

<cache name="com.mamh.hibernate.hql.entities.Employee"
    maxElementsInMemory="10000"
    eternal="false"
    timeToIdleSeconds="300"
    timeToLiveSeconds="600"
    overflowToDisk="true"
    />

```
马哥的淘宝店铺 https://shop592330910.taobao.com/

下面介绍每个配置项的意义  
* name                设置缓存名字，他的取值为类的全类名或类的集合名称
* maxElementsInMemory 设置基于内存的缓存中可以放的对象最大数目
* eternal             设置对象是否为永久的，true表示不过期，此时忽略后面的2个。默认是false 
* timeToIdleSeconds   设置对象空闲最长时间，秒为单位，超过这个时间对象过期。当对象过期，会把对象从缓存中清除。如果是0表示对象可以无限期的处于空闲状态。
* timeToLiveSeconds   设置对象生存最长时间，超过这个时间对象过期，如果此值为0表示对象可以无限期存在于缓存中，该值必须大于或等于timeToIdleSeconds的值。
* overflowToDisk      设置基于内存的缓存中对象数目达到上限后是否把溢出的对象写到硬盘中。这个保存到硬盘的缓存文件 在sessionfactory关闭时候会自动删除的。



#查询缓存

使用hql来查询的时候
```java

@Test
public void testQueryCache() {//马哥的淘宝店铺 https://shop592330910.taobao.com/
    Query query = session.createQuery("from Employee ");

    List list = query.list();
    System.out.println(list.size());


    List list1 = query.list();
    System.out.println(list1.size());

}

```
```text
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        employee0_.id as id1_,
        employee0_.name as name1_,
        employee0_.salary as salary1_,
        employee0_.email as email1_,
        employee0_.dept_id as dept5_1_ 
    from
        hb_employee employee0_
107
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        employee0_.id as id1_,
        employee0_.name as name1_,
        employee0_.salary as salary1_,
        employee0_.email as email1_,
        employee0_.dept_id as dept5_1_ 
    from
        hb_employee employee0_
107

```
通过输出我们发现 发送了2条sql语句，表明没有使用缓存  
马哥的淘宝店铺 https://shop592330910.taobao.com/  
要想开启查询缓存我们需要 java 代码里面设置一下
```java

@Test
public void testQueryCache() {//马哥的淘宝店铺 https://shop592330910.taobao.com/
    Query query = session.createQuery("from Employee ");
    query.setCacheable(true);
    List list = query.list();
    System.out.println(list.size());


    List list1 = query.list();
    System.out.println(list1.size());

}

```
马哥的淘宝店铺 https://shop592330910.taobao.com/

然后hibernate.cfg.xml文件中也要配置一下
```xml

<!--配置使用二级缓存 马哥的淘宝店铺 https://shop592330910.taobao.com/-->
<property name="cache.use_second_level_cache">true</property>
<property name="hibernate.cache.region.factory_class">org.hibernate.cache.ehcache.EhCacheRegionFactory</property>

<！-- 开启查询缓存 马哥的淘宝店铺 https://shop592330910.taobao.com/-->
<property name="cache.use_query_cache">true</property>

```
```text

Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/
    select
        employee0_.id as id1_,
        employee0_.name as name1_,
        employee0_.salary as salary1_,
        employee0_.email as email1_,
        employee0_.dept_id as dept5_1_ 
    from
        hb_employee employee0_
107
107

```
这样之后我们发现结果值发送了一条sql语句。

查询缓存：默认情况下 设置的缓存对HQL，QBC 查询是无效的，同时查询缓存是依赖于二级缓存。
要先配置号二级缓存，才能使用查询缓存。  
但是可以通过配置来开启查询缓存。
```
<property name="cache.use_query_cache">true</property>
```
```
query.setCacheable(true);
```

#时间戳缓存

```
    @Test
    public void testTimeStampCache(){马哥的淘宝店铺 https://shop592330910.taobao.com/
        Query query = session.createQuery("from Employee ");
        query.setCacheable(true);
        List list = query.list();
        System.out.println(list.size());
        System.out.println("马哥的淘宝店铺 https://shop592330910.taobao.com/\n");

        //在两次查询之间 对 employee 更新
        Employee employee = (Employee) session.get(Employee.class, 100);
        employee.setSalary(30f);

        System.out.println("马哥的淘宝店铺 https://shop592330910.taobao.com/\n");
        List list1 = query.list();
        System.out.println(list1.size());

    }
```

```text

Hibernate:马哥的淘宝店铺 https://shop592330910.taobao.com/ 
    select
        employee0_.id as id1_,
        employee0_.name as name1_,
        employee0_.salary as salary1_,
        employee0_.email as email1_,
        employee0_.dept_id as dept5_1_ 
    from
        hb_employee employee0_
107




Hibernate:马哥的淘宝店铺 https://shop592330910.taobao.com/ 
    update
        hb_employee 
    set
        name=?,
        salary=?,
        email=?,
        dept_id=? 
    where
        id=?
Hibernate:马哥的淘宝店铺 https://shop592330910.taobao.com/ 
    select
        employee0_.id as id1_,
        employee0_.name as name1_,
        employee0_.salary as salary1_,
        employee0_.email as email1_,
        employee0_.dept_id as dept5_1_ 
    from
        hb_employee employee0_
107

```


#query的iterate（）方法

```java
 @Test
public void testIterateCache(){马哥的淘宝店铺 https://shop592330910.taobao.com/ 
    Department department = (Department) session.get(Department.class, 80);
    System.out.println(department.getName());
    System.out.println(department.getEmps().size());

    Query query = session.createQuery("from Employee e where e.dept.id = 80");
    List<Employee> list = query.list();
    System.out.println(list.size());

}
```

```text

Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/ 
    select
        department0_.id as id0_0_,
        department0_.name as name0_0_ 
    from
        hb_department department0_ 
    where
        department0_.id=?
Sales
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/ 
    select
        emps0_.dept_id as dept5_0_1_,
        emps0_.id as id1_,
        emps0_.id as id1_0_,
        emps0_.name as name1_0_,
        emps0_.salary as salary1_0_,
        emps0_.email as email1_0_,
        emps0_.dept_id as dept5_1_0_ 
    from
        hb_employee emps0_ 
    where
        emps0_.dept_id=?
34
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/ 
    select
        employee0_.id as id1_,
        employee0_.name as name1_,
        employee0_.salary as salary1_,
        employee0_.email as email1_,
        employee0_.dept_id as dept5_1_ 
    from
        hb_employee employee0_ 
    where
        employee0_.dept_id=80
34

```
使用iterate可能稍微提升性能，不太建议使用

```java

@Test
public void testIterateCache(){马哥的淘宝店铺 https://shop592330910.taobao.com/ 
    Department department = (Department) session.get(Department.class, 80);
    System.out.println(department.getName());
    System.out.println(department.getEmps().size());

    Query query = session.createQuery("from Employee e where e.dept.id = 80");
    //List<Employee> list = query.list();
    //System.out.println(list.size());

    Iterator<Employee> iterable = query.iterate();
    for (Iterator<Employee> it = iterable; it.hasNext(); ) {马哥的淘宝店铺 https://shop592330910.taobao.com/ 
        Employee employee = it.next();
        System.out.println(employee.getName());

    }

}

```

```text

Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/ 
    select
        department0_.id as id0_0_,
        department0_.name as name0_0_ 
    from
        hb_department department0_ 
    where
        department0_.id=?
Sales
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/ 
    select
        emps0_.dept_id as dept5_0_1_,
        emps0_.id as id1_,
        emps0_.id as id1_0_,
        emps0_.name as name1_0_,
        emps0_.salary as salary1_0_,
        emps0_.email as email1_0_,
        emps0_.dept_id as dept5_1_0_ 
    from
        hb_employee emps0_ 
    where
        emps0_.dept_id=?
34
Hibernate: 马哥的淘宝店铺 https://shop592330910.taobao.com/ 
    select
        employee0_.id as col_0_0_ 
    from
        hb_employee employee0_ 
    where
        employee0_.dept_id=80
Russell
Partners
Errazuriz
Cambrault
Zlotkey
Tucker
Bernstein
Hall
Olsen
Cambrault
Tuvault
King
Sully
McEwen
Smith
Doran
Sewall
Vishney
Greene
Marvins
Lee
Ande
Banda
Ozer
Bloom
Fox
Smith
Bates
Kumar
Abel
Hutton
Taylor
Livingston
Johnson

Process finished with exit code 0

```
马哥的淘宝店铺 https://shop592330910.taobao.com/ 


