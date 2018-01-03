----------
Session概述

 - session接口是hibernate向应用程序提供的操作数据库的最主要接口，它提供了基本的保存，更新，删除和加载java对象的方法
 - session具有一个缓存，位于缓存中的对象称为持久化对象，他和数据库中的相关记录对应，session能够在某些时间点按照缓存中对象的变化来执行相关的sql语句，来同步更新数据库，这一过程被称为刷新缓存flush。
 - 站在持久化的角度，hibernate把对象分为4中状态，持久化状态，临时状态，游离状态，删除状态，session特定的方法能使对象从一个状态转换到另外一个状态。

马哥的淘宝店:https://shop592330910.taobao.com/

这里我们建立一个HibernateTest测试类。其中放置了Session和Transaction成员变量，这个在开发中不能放置为成员变量，会有并发问题的，这里我们只是测试可以放置一下。
我们通过单元测试的 @Before public void init()来初始化我们成员变量，然后通过@After    public void destroy() 来关闭。
```
public class HibernateTest {
    private SessionFactory sessionFactory = null;
    private Session session = null;
    private Transaction transaction = null;


    @Before
    public void init() {
        System.out.println("init");
        //1.创建一个SessionFactory 对象，创建session的工厂的一个类
        //1.1创建一个Configuration对象，对应hibernate的基本配置信息和对象关系映射信息
        Configuration configuration = new Configuration().configure();
        //1.2创建一个ServiceRegistry对象，hibernate4.x新添加的对象，hibernate任何的配置和服务都要在该对象中注册才能有效。
        ServiceRegistry serviceRegistry = new ServiceRegistryBuilder().applySettings(configuration.getProperties()).buildServiceRegistry();
        //1.3
        sessionFactory = configuration.buildSessionFactory(serviceRegistry);
        //2.创建一个Session对象,这个和jdbc中的connection很类似
        session = sessionFactory.openSession();
        //3.开启事务
        transaction = session.beginTransaction();
    }
    //马哥的淘宝店:https://shop592330910.taobao.com/
    @After
    public void destroy() {
        System.out.println("destroy");
        //5.提交事务
        transaction.commit();

        //6.关闭session对象
        session.close();

        //7.关闭SessionFactory 对象
        sessionFactory.close();
    //马哥的淘宝店:https://shop592330910.taobao.com/

    }

    @Test
    public void testSession() {
        //4.执行保存操作
        News news = new News("title:java", "author: mamh", new Date(new java.util.Date().getTime()));
        session.save(news);

        Object o = session.get(News.class, 1);
        System.out.println(o);
        //马哥的淘宝店:https://shop592330910.taobao.com/

    }
```


----------


session的缓存
```
    @Test
    public void testSession() {
        //4.执行保存操作
        //News news = new News("title:java", "author: mamh", new Date(new java.util.Date().getTime()));
        //session.save(news);

        Object news1 = session.get(News.class, 1);
        System.out.println(news1);

        Object news2 = session.get(News.class, 1);
        System.out.println(news2);

    }
```
这个代码的输出，从中我们可以看到news对象打印了两遍，但是sql语句只打印了一边，说明第二次没有去执行sql语句。这个是一级缓存。一级缓存是session级别的。
```
Hibernate: 
    select
        news0_.hb_id as hb1_0_0_,
        news0_.hb_title as hb2_0_0_,
        news0_.hb_author as hb3_0_0_,
        news0_.hb_date as hb4_0_0_ 
    from
        atguigu.hb_news news0_ 
    where
        news0_.hb_id=?
News{id=1, title='java', author='sun', date=2017-10-18}

News{id=1, title='java', author='sun', date=2017-10-18}

destroy

Process finished with exit code 0
```


----------


session缓存的flush（）方法

```
    @Test
    public void testSessionFlush(){
        News news = (News) session.get(News.class, 1);
        System.out.println(news);
        news.setAuthor("mamh");
    }
```

```
Hibernate: 
    select
        news0_.hb_id as hb1_0_0_,
        news0_.hb_title as hb2_0_0_,
        news0_.hb_author as hb3_0_0_,
        news0_.hb_date as hb4_0_0_ 
    from
        atguigu.hb_news news0_ 
    where
        news0_.hb_id=?
News{id=1, title='java', author='mamh', date=2017-10-18}

=destroy=
Hibernate: 
    update
        atguigu.hb_news 
    set
        hb_title=?,
        hb_author=?,
        hb_date=? 
    where
        hb_id=?

Process finished with exit code 0
```
在执行`(News) session.get(News.class, 1)`  会调用select查询数据库的，后面的`news.setAuthor("mamh");`  设置对象一个新的属性，这个时候会调用update的更新数据库的方法的。
这个是因为session发现缓存中的对象被更改了，然后会调用flush（）。修改对象属性的时候session的可以感知到的，在提交事务之前会执行update方法的。在调用commit（）方法的时候会先进行flush，然后在执行commit。
flush（）的作用：数据库表中的记录和session缓存中的对象保持一致。为了保持一致有可能会发送对应的sql语句。对象状态一模一样是不会发送sql语句的。
在`transaction.commit();`  中的commit（）方法中先调用session的flush（）然后在调用commit（）。flush（）方法可能会发送sql语句，但是不会提交事务的，只有提交事务之后数据库表中的数据才会变化。

flush（）什么时候会被调用：
注意：在未提交事务或显示的调用flush（）方法之前也有可能会进行flush操作的。

 - 执行HQL或QBC查询，会先进行flush操作，以得到数据表的最新的记录。
 - 若记录的ID是有底层数据库使用自增方式生成的，则调用save（）之后就会立即发送insert语句，save之后必须保证对象的id是存在的。


----------


session缓存的refresh()方法
refresh()方法会强制发送select语句，以使session缓存中对象的状态和数据库表中对应的保持一致。


----------


session缓存的clear()方法
清理缓存

```

    @Test
    public void testClear(){
        News news0 = (News) session.get(News.class, 1);

        session.clear();


        News news1 = (News) session.get(News.class, 1);

    }
```

```
Hibernate: 
    select
        news0_.hb_id as hb1_0_0_,
        news0_.hb_title as hb2_0_0_,
        news0_.hb_author as hb3_0_0_,
        news0_.hb_date as hb4_0_0_ 
    from
        atguigu.hb_news news0_ 
    where
        news0_.hb_id=?
Hibernate: 
    select
        news0_.hb_id as hb1_0_0_,
        news0_.hb_title as hb2_0_0_,
        news0_.hb_author as hb3_0_0_,
        news0_.hb_date as hb4_0_0_ 
    from
        atguigu.hb_news news0_ 
    where
        news0_.hb_id=?
```
第一次调用获取new对象的时候会发送一条select查询语句的，然后调用了session的clear（）方法，清空缓存了，第二次在调用获取new对象的时候会再次发送一条select查询语句。


----------


session中的save()和perssit()方法

```
    @Test
    public void testPersist() {
        //和save()方法很类似,区别：
        //在调用persist方法之前，调用了setId(),对象已经有了ID了，则不会执行insert操作，会抛出异常
        News news = new News();
        news.setAuthor("mm");
        news.setTitle("ssssssssss");
        news.setDate(new Date(new java.util.Date().getTime()));
        news.setId(234234);
        session.persist(news);
    }

    @Test
    public void testSave() {
        //把临时对象变为持久化对象
        //为对象分配ID,在save()方法之前设置ID是无效的，save()之后也是不能改这个ID的
        //持久化的对象的ID是不能进行修改的
        //在flush缓存时候会发送一条insert语句
        //把临时对象保存到数据库中
        News news = new News();
        news.setAuthor("mm");
        news.setTitle("ssssssssss");
        news.setDate(new Date(new java.util.Date().getTime()));

        session.save(news);
    }
```


----------


session中的get()和load()方法
load和get的区别：
get会立即加载对象，load方法若不用该对象则不会立即查询，而返回一个代理对象。
get立即检索，load是延迟检索。
若数据表中没有对应的记录,session也没有关闭,同时要使用该对象时候：get返回null，load抛出异常。
load方法可能会抛出懒加载异常：
org.hibernate.LazyInitializationException: could not initialize proxy - no Session
在需要初始化代理对象之前关闭了session就会抛出这个异常。
```
@Test
    public void testLoad(){
        //load和get的区别：
        //get会立即加载对象，load方法若不用该对象则不会立即查询，而返回一个代理对象。
        //get立即检索，load是延迟检索。
        //若数据表中没有对应的记录,session也没有关闭,同时要使用该对象时候：get返回null，load抛出异常。
        //load方法可能会抛出懒加载异常：
        // org.hibernate.LazyInitializationException: could not initialize proxy - no Session
        // 在需要初始化代理对象之前关闭了session就会抛出这个异常。
        News news1 = (News) session.load(News.class, 10);
        //session.close();
        System.out.println(news1);
    }

    @Test
    public void testGet(){
        News news1 = (News) session.get(News.class, 1);
        System.out.println(news1);
    }
```
----------
session的update()方法
若更新一个持久化对象，不需要显示的调用update()方法,因为在调用transaction的commit方法时候，会先执行flush方法的。
更新一个游离对象，需要显示的调用session的update()方法，可以把游离对象变为持久化对象。
无论要更新的对象 是否和数据库表中的记录是否一致都会发送update语句的。       
```
    @Test
    public void testUpdate(){
        //若更新一个持久化对象，不需要显示的调用update()方法,因为在调用transaction的commit方法时候，会先执行flush方法的。
        //更新一个游离对象，需要显示的调用session的update()方法，可以把游离对象变为持久化对象。
        //无论要更新的对象 是否和数据库表中的记录是否一致都会发送update语句的。因为是第二次开的一个新的session。
        News news = (News) session.get(News.class, 1);

        transaction.commit();
        session.close();

        session = sessionFactory.openSession();
        transaction = session.beginTransaction();
        //这个时候news对象已经变成游离对象了，应为前面关闭了session，后续又重新打开了session。

        news.setAuthor("sun");
        System.out.println(news);
        //session.update(news);
    }
```

```
Hibernate: 
    select
        news0_.hb_id as hb1_0_0_,
        news0_.hb_title as hb2_0_0_,
        news0_.hb_author as hb3_0_0_,
        news0_.hb_date as hb4_0_0_ 
    from
        atguigu.hb_news news0_ 
    where
        news0_.hb_id=?
News{id=1, title='java', author='oracle', date=2017-10-18}

Hibernate: 
    update
        atguigu.hb_news 
    set
        hb_title=?,
        hb_author=?,
        hb_date=? 
    where
        hb_id=?
```
如何让update盲目的发送update语句呢？在.hbm.xml 中设置select-before-update="true"（默认false），通常不设置为true，会影响效率。
若数据库表中没有对应的记录还继续调用update()方法，这时候会抛出异常。
```
    @Test
    public void testUpdate(){
        //若更新一个持久化对象，不需要显示的调用update()方法,因为在调用transaction的commit()方法时候，会先执行flush()方法的。
        //更新一个游离对象，需要显示的调用session的update()方法，可以把游离对象变为持久化对象。
        //无论要更新的对象 是否和数据库表中的记录是否一致都会发送update语句的
        //如何让update盲目的发送update语句呢？在.hbm.xml 中设置select-before-update="true"（默认false），通常不设置为true，会影响效率。
        //若数据库表中没有对应的记录还继续调用update()方法，这时候会抛出异常。

        News news = (News) session.get(News.class, 1);

        transaction.commit();
        session.close();

        session = sessionFactory.openSession();
        transaction = session.beginTransaction();
        //这个时候news对象已经变成游离对象了，应为前面关闭了session，后续又重新打开了session。

        news.setAuthor("sun");
        news.setId(120);
        System.out.println(news);
        session.update(news);
    }
```
同一个session中不能有ID相同的对象,会抛出异常NonUniqueObjectException

```
    @Test
    public void testUpdate(){
        //若更新一个持久化对象，不需要显示的调用update()方法,因为在调用transaction的commit方法时候，会先执行flush方法的。
        //更新一个游离对象，需要显示的调用session的update()方法，可以把游离对象变为持久化对象。
        //无论要更新的对象 是否和数据库表中的记录是否一致都会发送update语句的
        //如何让update盲目的发送update语句呢？在.hbm.xml 中设置select-before-update="true"（默认false），通常不设置为true，会影响效率。
        //若数据库表中没有对应的记录还继续调用update()方法，这时候会抛出异常。
        //同一个session中不能有ID相同的对象,会抛出异常NonUniqueObjectException
        News news = (News) session.get(News.class, 1);

        transaction.commit();
        session.close();

        session = sessionFactory.openSession();
        transaction = session.beginTransaction();
        //这个时候news对象已经变成游离对象了，应为前面关闭了session，后续又重新打开了session。

        News news2 = (News) session.get(News.class, 1);
        news.setAuthor("sun");
        session.update(news);
    }
```
----------
session的saveOrUpdate()方法

```
    @Test
    public void testSaveOrUpdate() {
        News news = new News("ff", "fff", new Date(new java.util.Date().getTime()));
        session.saveOrUpdate(news);
    }
```
这个没有ID，这个时候会执行insert语句。
```
Hibernate: 
    insert 
    into
        atguigu.hb_news
        (hb_title, hb_author, hb_date) 
    values
        (?, ?, ?)
```

如果有ID：会执行update语句

```
    @Test
    public void testSaveOrUpdate() {
        News news = new News("ff", "fff", new Date(new java.util.Date().getTime()));
        news.setId(1);
        session.saveOrUpdate(news);
    }
```

```
Hibernate: 
    select
        news_.hb_id,
        news_.hb_title as hb2_0_,
        news_.hb_author as hb3_0_,
        news_.hb_date as hb4_0_ 
    from
        atguigu.hb_news news_ 
    where
        news_.hb_id=?
Hibernate: 
    update
        atguigu.hb_news 
    set
        hb_title=?,
        hb_author=?,
        hb_date=? 
    where
        hb_id=?
```

如果有id，但是没有对应的记录，这个时候会抛出异常

```

    @Test
    public void testSaveOrUpdate() {
        News news = new News("ff", "fff", new Date(new java.util.Date().getTime()));
        news.setId(23121);
        session.saveOrUpdate(news);
    }
```
----------
session的delete方法
执行删除操作，只要OID和数据库表中的记录对应就会执行删除操作，如果没有对应的记录就抛出异常。
```
    @Test
    public void testDelete() {
        //执行删除操作，只要OID和数据库表中的记录对应就会执行删除操作，如果没有对应的记录就抛出异常。
        News news = new News("ff", "fff", new Date(new java.util.Date().getTime()));
        news.setId(1);
        session.delete(news);
    }
```

```
Hibernate: 
    delete 
    from
        atguigu.hb_news 
    where
        hb_id=?
```
可以通过设置hibernate配置文件use_identifier_rollback属性为true，是删除对象后，把其OID置为null

```
@Test
public void testDelete() {
    //执行删除操作，只要OID和数据库表中的记录对应就会执行删除操作，如果没有对应的记录就抛出异常。
    //可以通过设置hibernate配置文件use_identifier_rollback属性为true，是删除对象后，把其OID置为null
    News news = new News(8, "ff", "fff", new Date(new java.util.Date().getTime()));
    System.out.println(news);
    session.delete(news);
    System.out.println(news);
}
```
session的evict()方法，从缓存中移除某个对象
```
    @Test
    public void testEvict(){
        News news1 = (News) session.get(News.class, 1);
        news1.setAuthor("11");

        News news2 = (News) session.get(News.class, 2);
        news2.setAuthor("==============");

        // session.evict(news1);//加了这一句就不会更新到数据库中了
    }
```


----------


new 语句   --> 临时对象
                |
                |
                持久化对象
                |
                |
                |
                游离状态


----------
通过hibernate调用存储过程

```
    @Test
    public void testDoWork(){
        session.doWork(new Work() {
            public void execute(Connection connection) throws SQLException {
                //在这个里面调用存储过程
                System.out.println(connection);
            }
        });
    }
```

```
输出：
org.hibernate.engine.jdbc.internal.proxy.ConnectionProxyHandler@7526515b[valid=true]
```

hibernate与触发器协同工作(了解)
hibernate与触发器协同工作时会造成两类问题
>触发器使session的缓存中的持久化对象与数据库中对应的数据不一致，触发器运行在数据库中，它执行的操作对session是透明的。
>session的update方法盲目的激发触发器，无论游离对象的属性是否发生变化，都会执行update语句，而update语句会激发数据库中相应的触发器

解决方法：
>在执行完session的相关操作后，立即调用session的flush和refresh方法，迫使session的缓存与数据库同步
>在映射文件的class元素中设置select-before-update属性，当session的update或saveOrUpdate方法更新一个游离对象时，会先执行select语句，获得当前游离对像在数据库中的最新数据，只有在不一致的情况下才会执行update语句。
