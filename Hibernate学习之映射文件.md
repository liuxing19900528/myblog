Hibernate学习之映射文件

 pojo映射文件
https://shop592330910.taobao.com/
```
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping>

    <class name="com.mamh.hibernate.demo.entities.News" table="hb_news" schema="mage" select-before-update="true">
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
            <column name="hb_date" sql-type="datetime" not-null="true"/>
        </property>
    </class>
</hibernate-mapping>
```


----------
hibernate-mapping节点 中的package属性
```
<hibernate-mapping package="com.mamh.hibernate.demo.entities">

```
这个指定了，在class节点中的name属性就不用写全类名了。


----------
class节点，class节点在hibernate-mapping下面。
一个hibernate-mapping可以写多个class节点，不过一般都是一个。一个class一个xml文件。
```
    <class  name="News" table="hb_news" schema="mage" select-before-update="true">
```

 - name  全类名
 - table 数据库中的对应的表名
 - dynamic-insert  这个和动态更新类似
 - dynamic-update动态更新
 - select-before-update="true"，设置hibernate在更新某个持久化对象之前是否需要先执行一次查询，默认是false。建议设置false。
 

```
    @Test
    public void testDynamicUpdate(){
        News news = (News) session.get(News.class, 1);
        news.setAuthor("xxxxxxxxxxxxxxxx");
    }
如果class下不设置dynamic-update动态更新为true，上面的更新操作会执行如下的sql语句
    Hibernate: 
    update
        mage.hb_news 
    set
        hb_title=?,
        hb_author=?,
        hb_date=? 
    where
        hb_id=?
    这个sql语句会更新其他的不相关的属性。
如果class 设置dynamic-update动态更新为true，只会更新设置的那一个属性
    @Test
    public void testDynamicUpdate(){
        News news = (News) session.get(News.class, 1);
        news.setAuthor("dfsdfasdf");
    }
输出如下：
    Hibernate: 
        update
            mage.hb_news 
        set
            hb_author=? 
        where
            hb_id=?


```


----------
id节点，id节点在class节点里面
```
<id name="id">
    <column name="hb_id" sql-type="int(11)"/>
    <!--指定主键的生成方法，native使用数据库本地方式，mysql就是自动增长的数字-->
    <generator class="native"/>
</id>


```
hibernate使用对象标示符（OID）来建立内存中的对象和数据库表中记录的对应关系，对象的OID和数据表的主键对应。
hibernate通过标识符生成器来为主键赋值。

hibernate推荐数据表中使用代理主键（即不具备业务含义的字段，代理主键通常为整型，因为整型比字符串节省数据库空间）。

其中id元素用来设置对象标识符，generator子元素用来设定标识符生成器。

hibernate提供了标识符生成器接口，IdentifierGenerator，并提供了各种内置实现。





id节点中的 generator子节点

 - increment属性，由hibernate以递增方式为代理主键赋值。测试可以使用，正式开发不能用。
```
    @Test
    public void testIdGenerator() {
        News news = new News("ff", "fff", new Date(new java.util.Date().getTime()));
        session.save(news);

    }
Hibernate: 
    select
        max(hb_id) 
    from
        hb_news
=destroy=
Hibernate: 
    insert 
    into
        mage.hb_news
        (hb_title, hb_author, hb_date, hb_id) 
    values
        (?, ?, ?, ?)
```

 - identity，由底层数据库来负责生成。mysql可以使用这个。
 - sequence，利用底层数据库提供的序列来生成主键。DB2，oracle支持。
 - hilo，有hibernate安装一种high/low算法生成主键，这种方式不依赖任何数据库。都支持。

```
Hibernate: 
    select
        next_hi 
    from
        hibernate_unique_key for update
            
Hibernate: 
    update
        hibernate_unique_key 
    set
        next_hi = ? 
    where
        next_hi = ?
=destroy=
Hibernate: 
    insert 
    into
        mage.hb_news
        (hb_title, hb_author, hb_date, hb_id) 
    values
        (?, ?, ?, ?)

Process finished with exit code 0
```

 - native，依据数据库对自动生成主键的支持方式自动选择


----------


property节点，在class节点下面
https://shop592330910.taobao.com/
```
<property name="title">
    <column name="hb_title" sql-type="varchar(1024)" length="1024" not-null="true"/>
</property>
<property name="author">
    <column name="hb_author" sql-type="varchar(2048)" length="2048" not-null="true"/>
</property>
<property name="date">
    <column name="hb_date" sql-type="datetime" not-null="true"/>
</property>
```
映射派生属性 
https://shop592330910.taobao.com/
```
<!-- 映射派生属性 -->
<property name="desc" formula="(select concat(hb_author, ':= ', hb_title) from hb_news)"/>

```



java中时间日期的映射

>java中代表时间日期的类型包括java.util.Date 和 java.util.Calendar
此外，jdbc api提供了3个扩展   java.util.Date 的子类java.sql.Date,java.sql.Time,
java.sql.Timestamp,这个3个类分别和标准的sql类型的date，time，timestamp类型对应。

>在标准sql中，DATE表示日期，TIME表示时间，TIMESTAMP表示时间戳，同时包含日期和时间。

>java.util.Date 可以对应3中类型，设计类的时候建议使用这个类型。

>如何把 java.util.Date 映射为DATE，TIME ，TIMESTAMP类型呢？可以通过property的type属性。
```
<property name="date" type="timestamp">
    <column name="hb_date"  />
</property>
```

映射大数据类型
```

<property name="content" type="clob">
    <column name="hb_content" sql-type="mediumtext"/>
</property>

<property name="image" type="blob">
    <column name="hb_image" sql-type="mediumblob"/>
</property>
```

```
   @Test
    public void testBlob() {
        News news = new News();
        news.setTitle("title");
        news.setAuthor("author");
        news.setDate(new java.util.Date());

        news.setDesc("desc.......");

        InputStream stream = null;
        try {
            stream = new FileInputStream("/home/mamh/马哥私房菜-8X8-02.jpg");
            Blob image = Hibernate.getLobCreator(session).createBlob(stream, stream.available());
            news.setImage(image);
        } catch (IOException e) {
            e.printStackTrace();
        }

        session.save(news);
    }
```

组件映射
```
    @Test
    public void testWorker() {
        Worker worker = new Worker();
        Pay pay = new Pay();
        pay.setMonthlyPay(12000);
        pay.setYearPay(170000);
        pay.setVacation(10);

        worker.setName("bright.ma");
        worker.setPay(pay);

        session.save(worker);
    }
```

```
    <class name="Worker" table="hb_work" schema="mage" select-before-update="true" dynamic-update="true">
        <id name="id">
            <column name="hb_id" sql-type="int(11)"/>
            <!--指定主键的生成方法，native使用数据库本地方式，mysql就是自动增长的数字-->
            <generator class="hilo"/>
        </id>
        <property name="name" type="string" update="true" index="hb_title">
            <column name="hb_name" length="1024" not-null="true" unique="true"/>
        </property>


        <component name="pay">
            <property name="monthlyPay" column="hb_monthly_pay"/>
            <property name="yearPay" column="hb_year_pay"/>
            <property name="vacation" column="hb_vacation"/>
        </component>


    </class>
```

```
public class Worker {
    private Integer id;
    private String name;

    private Pay pay;
worker类里面有个其他类pay。

public class Pay {
    private int monthlyPay;
    private int yearPay;
    private int vacation;



```
