----------


马哥的淘宝店:https://shop592330910.taobao.com/



什么是Hibernate

 1. 一个框架
 2. 一个java领域持久化框架
 3. 一个ORM框架

什么是ORM
（object/relation mapping）对象关系映射
类  -- 表
对象--表的行
属性--表的列
将关系数据库表中的记录映射成为对象，以对象的形式展现，程序员可以把对数据库的操作转化为对对象的操作。
orm采用元数据来描述对象关系映射细节，元数据采用xml格式，存放在对象关系映射文件中。

马哥的淘宝店:https://shop592330910.taobao.com/

hibernate与jdbc代码对比

```
//这个是hibernate保存一个对象message的代码
public void save(Session sess, Message m){
    sess.save(m);
}

//下面是使用jdbc保存一个对象message的代码
public void save(Connection conn, Message m){
    PreparedStatement ps=null;
    String sql = "insert into message values(?, ?)";
    try{
        ps = conn.prepareStatement(sql);
        ps.setString(1, m.getTitle());
        ps.setString(2, m.getContent());
        ps.execute();
    }catch(){
        e.printStackTrace();
    }finally{
        if (ps != null){
            ps.close();
        }

    }
}

```
马哥的淘宝店:https://shop592330910.taobao.com/

hibernate开发步骤

1.通过hibernate  api编写访问数据库的代码
 

```
    @Test
    public void test() {
        //1.创建一个SessionFactory 对象，创建session的工厂的一个类
        SessionFactory sessionFactory = null;
        //1.1创建一个Configuration对象，对应hibernate的基本配置信息和对象关系映射信息
        Configuration configuration = new Configuration().configure();
        //1.2创建一个ServiceRegistry对象，hibernate4.x新添加的对象，hibernate任何的配置和服务都要在该对象中注册才能有效。
        ServiceRegistry serviceRegistry = new ServiceRegistryBuilder().applySettings(configuration.getProperties()).buildServiceRegistry();
        //1.3
        sessionFactory = configuration.buildSessionFactory(serviceRegistry);


        //2.创建一个Session对象
        Session session = sessionFactory.openSession();


        //3.开启事务
        Transaction transaction = session.beginTransaction();

        //4.执行保存操作
        News news = new News("title:java", "author: mamh", new Date(new java.util.Date().getTime()));
        session.save(news);

        //5.提交事务
        transaction.commit();

        //6.关闭session对象
        session.close();

        //7.关闭SessionFactory 对象
        sessionFactory.close();

    }
```
马哥的淘宝店:https://shop592330910.taobao.com/

2.创建持久化类
 必须提供一个无参数的构造器，因为需要用的反射的机制。

```
package com.mamh.hibernate.demo;

import java.sql.Date;

public class News {
    private Integer id;
    private String title;
    private String author;

    private Date date;


    public News() {//必须提供一个无参数的构造器
    }

    public News(String title, String author, Date date) {
        this.title = title;
        this.author = author;
        this.date = date;
    }

    public News(Integer id, String title, String author, Date date) {
        this.id = id;
        this.title = title;
        this.author = author;
        this.date = date;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getTitle() {
        return title;
    }

    public void setTitle(String title) {
        this.title = title;
    }

    public String getAuthor() {
        return author;
    }

    public void setAuthor(String author) {
        this.author = author;
    }

    public Date getDate() {
        return date;
    }

    public void setDate(Date date) {
        this.date = date;
    }

    @Override
    public String toString() {
        return "News{" +
                "id=" + id +
                ", title='" + title + '\'' +
                ", author='" + author + '\'' +
                ", date=" + date +
                '}' + '\n';
    }
}

```
马哥的淘宝店:https://shop592330910.taobao.com/

3.创建对象关系映射文件

```
<class name="com.mamh.hibernate.demo.News" table="hb_news" schema="atguigu">
指定类名和数据库表名的对应
```

```
<id name="id">
    <column name="hb_id" sql-type="int(11)"/>
    <!--指定主键的生成方法，native使用数据库本地方式，mysql就是自动增长的数字-->
    <generator class="native"/>
</id>
```

```
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping>

    <class name="com.mamh.hibernate.demo.News" table="hb_news" schema="atguigu">
        <id name="id">
            <column name="hb_id" sql-type="int(11)"/>
            <!--指定主键的生成方法，native使用数据库本地方式，mysql就是自动增长的数字-->
            <generator class="native"/>
        </id>
        <property name="title">
            <column name="hb_title" sql-type="varchar(1024)" length="1024" not-null="true"/>
        </property>
        <property name="author">
            <column name="hb_author" sql-type="varchar(2048)" length="2048" not-null="true"/>
        </property>
        <property name="date">
            <column name="hb_date" sql-type="date" not-null="true"/>
        </property>
    </class>
</hibernate-mapping>
```
马哥的淘宝店:https://shop592330910.taobao.com/

4.创建hibernate配置文件
这个配置文件的默认名是/hibernate.cfg.xml

```
通过configure()方法中的代码可以看到这个默认的配置文件名称。
    public Configuration configure() throws HibernateException {
        configure( "/hibernate.cfg.xml" );
        return this;
    }
```

```
<!--是否打印sql语句-->
<property name="show_sql">true</property>

<!--是否对sql语句格式化-->
<property name="format_sql">true</property>
配置了这2个属性就会有下面的输出。方便调试sql语句。
Hibernate: 
    insert 
    into
        atguigu.hb_news
        (hb_title, hb_author, hb_date) 
    values
        (?, ?, ?)
```

```
<!--指定自动生成数据库表的策略，有4个值-->
<property name="hbm2ddl.auto">update</property>```
update,数据库表个原来一样就不动，
create,每次都创建新的数据库表
create_drop,每次创建，用完删除表
validate
```


```
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>
        <!--  链接数据库的基本信息 -->
        <property name="hibernate.connection.username">test</property>
        <property name="hibernate.connection.password">123456</property>
        <property name="hibernate.connection.driver_class">com.mysql.jdbc.Driver</property>
        <property name="hibernate.connection.url">jdbc:mysql://10.0.63.43/test</property>

        <!-- 配置数据库的方言，注意这个有的版本好像不一样的。视频中使用的是这个MySQLInnoDBDialect，不过我直接照着做的时候好像这个值已经废弃，可能是hibernate/mysql的jar包版本问题把。[Maven: org.hibernate:hibernate-core:4.1.1.Final] org.hibernate.dialect中好像有个这个值。 -->
        <property name="dialect">org.hibernate.dialect.MySQL5Dialect</property>

        <!--是否打印sql语句-->
        <property name="show_sql">true</property>

        <!--是否对sql语句格式化-->
        <property name="format_sql">true</property>

        <!--指定自动生成数据库表的策略，有4个值-->
        <property name="hbm2ddl.auto">update</property>

        <!-- 这里使用的路径 -->
        <mapping resource="News.hbm.xml"/>
        
    </session-factory>
</hibernate-configuration>
```
maven. pom.xml 文件，使用maven来管理依赖的jar包文件。用idea新建一个空的maven工程，然后使用下面的pom.xml 内容就可以了。

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>hibernate</groupId>
    <artifactId>mamh</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <!-- https://mvnrepository.com/artifact/org.hibernate/hibernate-core -->
        <dependency>
            <groupId>org.hibernate</groupId>
            <artifactId>hibernate-core</artifactId>
            <version>4.1.1.Final</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>6.0.6</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/org.apache.logging.log4j/log4j-core -->
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>2.9.1</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/commons-dbutils/commons-dbutils -->
        <dependency>
            <groupId>commons-dbutils</groupId>
            <artifactId>commons-dbutils</artifactId>
            <version>1.7</version>
        </dependency>
        <!-- https://mvnrepository.com/artifact/com.mchange/c3p0 -->
        <dependency>
            <groupId>com.mchange</groupId>
            <artifactId>c3p0</artifactId>
            <version>0.9.5.2</version>
        </dependency>



        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>

    </dependencies>

</project>
```
马哥的淘宝店:https://shop592330910.taobao.com/



