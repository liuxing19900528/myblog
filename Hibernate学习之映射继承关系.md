hibernate支持三种继承映射策略：

使用subclass进行映射
>将域模型中的每一个实体对象映射到一个独立的表中，也就是说不用在关系数据模型中考虑域模型中的继承和多态。

使用joined-subclass进行映射
>对于继承关系中的子类使用通一个表，这就是需要在数据库表中增加额外的区分子类类型的字段。

使用union-subclass进行映射
>域模型中的每个类型映射到一个表，通过关系数据模型中的外键来描述表之间的继承关系。
>这也就相当于按照域模型的结构来建立关系数据库中的表，并通过外键来建立表之间的继承关系。


----------

1.使用subclass进行映射
>将域模型中的每一个实体对象映射到一个独立的表中，也就是说不用在关系数据模型中考虑域模型中的继承和多态。

person父类马哥的淘宝店:https://shop592330910.taobao.com/
```
public class Person {
    private Integer id;
    private String name;
    private int age;

    public Integer getId() {
        return id;马哥的淘宝店:https://shop592330910.taobao.com/
    }

    public void setId(Integer id) {
        this.id = id;马哥的淘宝店:https://shop592330910.taobao.com/
    }

    public String getName() {
        return name;马哥的淘宝店:https://shop592330910.taobao.com/
    }

    public void setName(String name) {
        this.name = name;马哥的淘宝店:https://shop592330910.taobao.com/
    }

    public int getAge() {
        return age;马哥的淘宝店:https://shop592330910.taobao.com/
    }

    public void setAge(int age) {
        this.age = age;马哥的淘宝店:https://shop592330910.taobao.com/
    }
}
```
学生子类
```
public class Student extends Person {
    private String school;

    public String getSchool() {
        return school;马哥的淘宝店:https://shop592330910.taobao.com/
    }

    public void setSchool(String school) {
        this.school = school;马哥的淘宝店:https://shop592330910.taobao.com/
    }
}

```
这里只需要一个person类的hbm映射文件
映射文件中需要添加对应的映射，该模型中只需要添加一个映射文件，因为只生成一张表，
在映射文件中添加对应的子类映射，使用subclass标签，标签中添加鉴别器discriminator-value，
该鉴别器属性指明了在数据库中写入数据时指示写入的是何种类型
```
<hibernate-mapping package="com.mamh.hibernate.demo.entities">

    <class name="Person" table="hb_person" schema="mage" discriminator-value="person">
        <id name="id">
            <column name="hb_id"/>
            <generator class="native"/>
        </id>马哥的淘宝店:https://shop592330910.taobao.com/

        <discriminator type="java.lang.String" column="type"/>

        <property name="name" type="java.lang.String">
            <column name="hb_name"/>
        </property>

        <property name="age" type="int">
            <column name="hb_age"/>
        </property>马哥的淘宝店:https://shop592330910.taobao.com/


        <!--映射子类 使用subclass-->

        <subclass name="Student" discriminator-value="student">
            <property name="school" type="java.lang.String">
                <column name="hb_school"/>
            </property>马哥的淘宝店:https://shop592330910.taobao.com/
        </subclass>


    </class>
</hibernate-mapping>
```


----------
插入操作
马哥的淘宝店:https://shop592330910.taobao.com/
```
    @Test
    public void testSavePerson(){
        //插入操作对于子类对象只需包记录擦人到一张表中。
        // 辨别者列由hibernate自动维护。
        Person person = new Person();
        person.setAge(18);
        person.setName("pppp");

        Student student = new Student() ;
        student.setSchool("zz");
        student.setAge(128);
        student.setName("sss");

        session.save(person);
        session.save(student);
    }
```

```

Hibernate: 
    insert 
    into
        mage.hb_person
        (hb_name, hb_age, type) 
    values
        (?, ?, 'person')
Hibernate: 马哥的淘宝店:https://shop592330910.taobao.com/
    insert 
    into
        mage.hb_person
        (hb_name, hb_age, hb_school, type) 
    values
        (?, ?, ?, 'student')
=destroy=马哥的淘宝店:https://shop592330910.taobao.com/

Process finished with exit code 0
```
对于插入操作，只需要把记录插入到一张数据表中。辨别者列由hibernate自动维护。

|#|hb_id|type|hb_name|hb_age|hb_school|
|-|----|----|----|---|
|1|1|person|pppp|18|NULL|
|2|2|student|sss|128|zz|
|3|3|person|pppp|18|NULL|
|4|4|student|sss|128|zz|
|5|5|person|pppp|18|NULL|
|6|6|student|sss|128|zz|




----------
查询操作
```
    @Test
    public void testGetPerson(){
        List person = session.createQuery("from Person ").list();
        System.out.println(person);

        List student = session.createQuery("from Student ").list();
        System.out.println(student);
    }
```
查询操作：父类记录，只需要一张数据表。对于子类记录，也只需要一张数据表，自动的给加上type=“student”
```
Hibernate: 
    select
        person0_.hb_id as hb1_9_,
        person0_.hb_name as hb3_9_,
        person0_.hb_age as hb4_9_,
        person0_.hb_school as hb5_9_,
        person0_.type as type9_ 
    from
        mage.hb_person person0_
        
[
    Person{id=1, name='pppp', age=18}, 
    Person{id=2, name='sss', age=128}, 
    Person{id=3, name='pppp', age=18}, 
    Person{id=4, name='sss', age=128}
]

Hibernate: 
    select
        student0_.hb_id as hb1_9_,
        student0_.hb_name as hb3_9_,
        student0_.hb_age as hb4_9_,
        student0_.hb_school as hb5_9_ 
    from
        mage.hb_person student0_ 
    where
        student0_.type='student'
[
    Person{id=2, name='sss', age=128}, 
    Person{id=4, name='sss', age=128}
]
=destroy=

Process finished with exit code 0

```
查询student子类的时候我们可以看到 其中的sql语句是有个student0_.type='student'这个条件的。
这个type这一列就是辨别者列。

缺点：
使用了辨别者列。
子类独有的字段不能添加非空约束。
若继承层次较深，则数据表的字段也会比较多。


----------
2.使用joined-subclass进行映射
>对于继承关系中的子类使用通一个表，这就是需要在数据库表中增加额外的区分子类类型的字段。
>
>采用joined-subclass元素的继承映射可以实现每个子类一张表。
>
>采用这种映射策略时，父类实例保存在父类表中，子类实例有父类表和子类表共同存储。
>因为子类实例也是一个特殊的父类实例，因此也包含了父类实例的属性，于是将子类和父类共有的
>属性保存在父类表中，子类独有的属性则保存在子类表中。
>
>在这种映射策略下，无需使用鉴别者列，但需要为每个子类使用key元素映射共有主键。
>
>子类增加的属性可以添加非空约束，因为子类的属性和父类的属性没有保存在同一个表中。  

```
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping package="com.mamh.hibernate.demo.entities">

    <class name="Person" table="hb_person" schema="mage" >
        <id name="id">
            <column name="hb_id"/>
            <generator class="native"/>
        </id>


        <property name="name" type="java.lang.String">
            <column name="hb_name"/>
        </property>

        <property name="age" type="int">
            <column name="hb_age"/>
        </property>

        <joined-subclass name="Student" table="hb_student">
            <key column="hb_id"></key>
            <property name="school" type="java.lang.String">
                <column name="hb_school"/>
            </property>
        </joined-subclass>


    </class>
</hibernate-mapping>
```


----------

保存操作

```

    @Test
    public void testSavePerson(){
        //插入操作对于子类对象只需包记录擦人到一张表中。
        // 辨别者列由hibernate自动维护。
        Person person = new Person();
        person.setAge(18);
        person.setName("pppp");

        Student student = new Student() ;
        student.setSchool("zz");
        student.setAge(128);
        student.setName("sss");

        session.save(person);
        session.save(student);
    }
```

```
Hibernate: 
    insert 
    into
        mage.hb_person
        (hb_name, hb_age) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        mage.hb_person
        (hb_name, hb_age) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        hb_student
        (hb_school, hb_id) 
    values
        (?, ?)
=destroy=

Process finished with exit code 0
```
person表格
|person表格 |hb_id|hb_name|hb_age|
|----------|--|----|----|
|1|1|pppp|18|
|2|2|sss|128|

student表格
|student表格|hb_id|hb_school|
|----------|------|--------|
|1|2|zz|


----------
查询操作
```
    @Test
    public void testGetPerson(){
        List person = session.createQuery("from Person ").list();
        System.out.println(person);

        List student = session.createQuery("from Student ").list();
        System.out.println(student);
    }
```

```
Hibernate: 
    select
        person0_.hb_id as hb1_9_,
        person0_.hb_name as hb2_9_,
        person0_.hb_age as hb3_9_,
        person0_1_.hb_school as hb2_10_,
        case 
            when person0_1_.hb_id is not null then 1 
            when person0_.hb_id is not null then 0 
        end as clazz_ 
    from
        mage.hb_person person0_ 
    left outer join
        hb_student person0_1_ 
            on person0_.hb_id=person0_1_.hb_id
[Person{id=2, name='sss', age=128}, Person{id=1, name='pppp', age=18}]
Hibernate: 
    select
        student0_.hb_id as hb1_9_,
        student0_1_.hb_name as hb2_9_,
        student0_1_.hb_age as hb3_9_,
        student0_.hb_school as hb2_10_ 
    from
        hb_student student0_ 
    inner join
        mage.hb_person student0_1_ 
            on student0_.hb_id=student0_1_.hb_id
[Person{id=2, name='sss', age=128}]
=destroy=

Process finished with exit code 0
```

优点：
不需要使用鉴别者列
子类独有的字段能添加非空约束
没有冗余的字段

----------
3.使用union-subclass进行映射
>域模型中的每个类型映射到一个表，通过关系数据模型中的外键来描述表之间的继承关系。
>这也就相当于按照域模型的结构来建立关系数据库中的表，并通过外键来建立表之间的继承关系。

>采用union-subclass元素可以实现将每一个实体对象映射到一个独立的表中。

>子类增加的属性可以有非空约束，即父类实例的数据保存在父类表中，而子类实例的数据保存在子类表中。

>子类实例的数据仅保存在子类表中，而在父类表中没有任何记录
>在这种映射策略下，子类表的字段会比父类表的映射字段要多，因为子类表的字段等于父类表的字段，
>加子类增加属性的总和。
>在这种策略下，既不需要使用鉴别者列，也无需使用key元素来映射共有主键。
>使用union-subclass映射策略是不可以使用identity的主键生成策略的，因为同一类继承层次中所有实体类
>都需要使用同一个主键种子，即多个持久化实体对应的记录的主键应该是连续的，受此影响，
>也不该使用native主键生成策略，应为native会根据数据库来选择使用identity或sequence。 

```
org.hibernate.MappingException: Cannot use identity column key generation with <union-subclass> mapping for: com.mamh.hibernate.demo.entities.Student
```

```
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping package="com.mamh.hibernate.demo.entities">

    <class name="Person" table="hb_person" schema="mage" >
        <id name="id">
            <column name="hb_id"/>
            <generator class="hilo"/>
        </id>


        <property name="name" type="java.lang.String">
            <column name="hb_name"/>
        </property>

        <property name="age" type="int">
            <column name="hb_age"/>
        </property>

        <union-subclass name="Student" table="hb_student">
            <property name="school" type="java.lang.String">
                <column name="hb_school"/>
            </property>
        </union-subclass>

    </class>
</hibernate-mapping>
```


----------
插入操作

```
    @Test
    public void testSavePerson(){
        //插入操作对于子类对象只需包记录擦人到一张表中。
        // 辨别者列由hibernate自动维护。
        Person person = new Person();
        person.setAge(18);
        person.setName("pppp");

        Student student = new Student() ;
        student.setSchool("zz");
        student.setAge(128);
        student.setName("sss");

        session.save(person);
        session.save(student);
    }
```

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
        mage.hb_person
        (hb_name, hb_age, hb_id) 
    values
        (?, ?, ?)
Hibernate: 
    insert 
    into
        hb_student
        (hb_name, hb_age, hb_school, hb_id) 
    values
        (?, ?, ?, ?)

Process finished with exit code 0
```

|父类person表|hb_id|hb_name|hb_age|
|-|-|-|-|
|1|1|pppp|18|
|2|2|sss|128|

|子类student表|hb_id|hb_name|hb_age|hb_school|
|-|-|-|-|-|
|1|2|sss|128|zz|


----------
查询操作

```
    @Test
    public void testGetPerson(){
        List person = session.createQuery("from Person ").list();
        System.out.println(person);

        List student = session.createQuery("from Student ").list();
        System.out.println(student);
    }
```

```
Hibernate: 
    select
        person0_.hb_id as hb1_9_,
        person0_.hb_name as hb2_9_,
        person0_.hb_age as hb3_9_,
        person0_.hb_school as hb1_10_,
        person0_.clazz_ as clazz_ 
    from
        ( select
            hb_id,
            hb_name,
            hb_age,
            null as hb_school,
            0 as clazz_ 
        from
            mage.hb_person 
        union
        select
            hb_id,
            hb_name,
            hb_age,
            hb_school,
            1 as clazz_ 
        from
            hb_student 
    ) person0_
[Person{id=1, name='pppp', age=18}, Person{id=2, name='sss', age=128}]
Hibernate: 
    select
        student0_.hb_id as hb1_9_,
        student0_.hb_name as hb2_9_,
        student0_.hb_age as hb3_9_,
        student0_.hb_school as hb1_10_ 
    from
        hb_student student0_
[Person{id=2, name='sss', age=128}]
=destroy=

Process finished with exit code 0
```
无需使用辨别者列。
子类可以添加非空约束。
多了冗余字段。


----------
更新操作
```
    @Test
    public void testUpdatePerson(){
        String hql = "update Person p set p.age = 120";
        session.createQuery(hql).executeUpdate();
    }
```

```
Hibernate: 
    create temporary table if not exists HT_hb_person (hb_id integer not null) 
Hibernate: 
    insert  into  HT_hb_person select person0_.hb_id as hb_id  from
            ( select hb_id,  hb_name, hb_age, null as hb_school, 0 as clazz_  from mage.hb_person 
          union 
              select  hb_id, hb_name, hb_age,  hb_school,  1 as clazz_  from  hb_student  
            ) person0_
Hibernate: 
    update mage.hb_person set hb_age=120 
    where ( hb_id ) IN ( select  hb_id  from  HT_hb_person  )
Hibernate: 
    update mage.hb_person set hb_age=120 
    where (hb_id ) IN ( select hb_id  from HT_hb_person )
Hibernate: 
    update hb_student set  hb_age=120 
    where (hb_id) IN ( select hb_id from HT_hb_person)

Hibernate: 
    drop temporary table HT_hb_person

=destroy=

Process finished with exit code 0
```

若更新父类属性，则效率比较低
