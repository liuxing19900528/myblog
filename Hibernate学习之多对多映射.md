多对多关联关系

#单向多对多关联关系

2张表，这个时候需要第三章表来表示多对多关系。

```
public class Category {
    private Integer id;
    private String name;
    马哥的淘宝店:https://shop592330910.taobao.com/
    //单向多对多这里有个集合用来保存item对象。
    private Set<Item> items = new HashSet<Item>();

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }
    马哥的淘宝店:https://shop592330910.taobao.com/
    public String getName() {
        return name;
    }
    马哥的淘宝店:https://shop592330910.taobao.com/
    public void setName(String name) {
        this.name = name;
    }
    马哥的淘宝店:https://shop592330910.taobao.com/
    public Set<Item> getItems() {
        return items;
    }

    public void setItems(Set<Item> items) {
        this.items = items;    马哥的淘宝店:https://shop592330910.taobao.com/
    }
}
```

```
public class Item {    马哥的淘宝店:https://shop592330910.taobao.com/
    private Integer id;
    private String name;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;    马哥的淘宝店:https://shop592330910.taobao.com/
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

```
<hibernate-mapping package="com.mamh.hibernate.demo.entities">

    <class name="Category" table="hb_category" schema="mage">
        <id name="id">
            <column name="hb_id"/>
            <generator class="native"/>
        </id>    马哥的淘宝店:https://shop592330910.taobao.com/
        <property name="name" type="string">
            <column name="hb_name"/>
        </property>

        <set name="items" table="hb_category_item" ><!--"hb_category_item"是中间表 -->
            <key><column name="hb_category_id"/></key>
            
            <!-- 使用manay to many 指定多对多关联关系。
            column是执行set集合中的持久化类在中间表的外键列名。-->
            <many-to-many class="Item" column="hb_item_id"/>

    马哥的淘宝店:https://shop592330910.taobao.com/

    </class>
</hibernate-mapping>
```

```
<hibernate-mapping package="com.mamh.hibernate.demo.entities">

    <class name="Item" table="hb_item" schema="mage">
        <id name="id">
            <column name="hb_id"/>
            <generator class="native"/>
        </id>    马哥的淘宝店:https://shop592330910.taobao.com/
        <property name="name" type="string">
            <column name="hb_name"/>
        </property>

    </class>    马哥的淘宝店:https://shop592330910.taobao.com/
</hibernate-mapping>
```


----------
插入操作

```
    @Test
    public void testSaveCategory() {    马哥的淘宝店:https://shop592330910.taobao.com/
        Category category1 = new Category();
        category1.setName("c-aa");
        Category category2 = new Category();
        category2.setName("c-bb");

        Item item1 = new Item();    马哥的淘宝店:https://shop592330910.taobao.com/
        item1.setName("item-aa");
        Item item2 = new Item();
        item2.setName("item-aa");

        category1.getItems().add(item1);
        category1.getItems().add(item2);
        category2.getItems().add(item1);
        category2.getItems().add(item2);

        session.save(category1);    马哥的淘宝店:https://shop592330910.taobao.com/
        session.save(category2);
        session.save(item1);
        session.save(item2);    马哥的淘宝店:https://shop592330910.taobao.com/
    }
```

```
Hibernate:     马哥的淘宝店:https://shop592330910.taobao.com/
    insert into mage.hb_category (hb_name) values (?)
Hibernate: 
    insert into mage.hb_category (hb_name) values (?)
Hibernate: 
    insert  into mage.hb_item (hb_name) values (?)
Hibernate: 
    insert into mage.hb_item (hb_name) values(?)
=destroy=    马哥的淘宝店:https://shop592330910.taobao.com/
Hibernate: 
    insert into hb_category_item (hb_category_id, hb_item_id) values (?, ?)
Hibernate: 
    insert into hb_category_item (hb_category_id, hb_item_id) values (?, ?)
Hibernate: 
    insert into hb_category_item (hb_category_id, hb_item_id) values (?, ?)
Hibernate: 
    insert into hb_category_item (hb_category_id, hb_item_id) values (?, ?)

Process finished with exit code 0
```
插入操作时候，显示4个category，item的那四个insert 语句，后面还会有4个插入中间表的4条insert语句。


----------
查询操作

```
    @Test
    public void testGetCategory(){    马哥的淘宝店:https://shop592330910.taobao.com/
        Category category = (Category) session.get(Category.class, 1);
        System.out.println(category.getName());
    }
```

```
Hibernate:     马哥的淘宝店:https://shop592330910.taobao.com/
    select
        category0_.hb_id as hb1_6_0_,
        category0_.hb_name as hb2_6_0_ 
    from
        mage.hb_category category0_ 
    where
        category0_.hb_id=?
c-aa
=destroy=    马哥的淘宝店:https://shop592330910.taobao.com/

Process finished with exit code 0
```
查询操作我们发现如果只是查询category的话，item是不会取查询的，懒加载。

```
    @Test
    public void testGetCategory(){    马哥的淘宝店:https://shop592330910.taobao.com/
        Category category = (Category) session.get(Category.class, 1);
        System.out.println(category.getName());

        System.out.println(category.getItems().size());
    }
```

```
Hibernate:     马哥的淘宝店:https://shop592330910.taobao.com/
    select
        category0_.hb_id as hb1_6_0_,
        category0_.hb_name as hb2_6_0_ 
    from
        mage.hb_category category0_ 
    where
        category0_.hb_id=?
c-aa
Hibernate:     马哥的淘宝店:https://shop592330910.taobao.com/
    select
        items0_.hb_category_id as hb1_6_1_,
        items0_.hb_item_id as hb2_1_,
        item1_.hb_id as hb1_8_0_,
        item1_.hb_name as hb2_8_0_ 
    from
        hb_category_item items0_ 
    inner join
        mage.hb_item item1_ 
            on items0_.hb_item_id=item1_.hb_id 
    where
        items0_.hb_category_id=?
2
=destroy=    马哥的淘宝店:https://shop592330910.taobao.com/

Process finished with exit code 0
```
这个时候如果取访问 category中的items集合的话就会连带查询item表了。
使用了inner join内联的方式来关联中间表。

以上是单向的多对多关联关系。

----------
#双向多对多关联关系

双向关联关系就是在Item类中也加一个集合，来保存category的引用。
原来的category类保持不变。
```
public class Item {    马哥的淘宝店:https://shop592330910.taobao.com/
    private Integer id;
    private String name;

    private Set<Category> categories = new HashSet<Category>();

    public Set<Category> getCategories() {
        return categories;    马哥的淘宝店:https://shop592330910.taobao.com/
    }

    public void setCategories(Set<Category> categories) {
        this.categories = categories;
    }
```
然后是item.hbm.xml 中也设置一个set属性，和category正好是相对的。
```    
    马哥的淘宝店:https://shop592330910.taobao.com/
        <set name="categories" table="hb_category_item">
            <key>
                <column name="hb_item_id"/>
            </key>
            <many-to-many class="Category" column="hb_category_id"/>
        </set>
```

这个时候双向的需要在任意一端 的set中设置一个inverse = true，只让一端来维护关联关系。

```

        <set name="items" table="hb_category_item" inverse="true">
            <key>
                <column name="hb_category_id"/>
            </key>
            <!-- 使用manay to many 指定多对多关联关系。
            column是执行set集合中的持久化类在中间表的外键列名。 hb_item_id是中间表的列名。-->
            <many-to-many class="Item" column="hb_item_id"/>
        </set>
```
----------
插入操作
```
    @Test
    public void testSaveCategory() {
        Category category1 = new Category();
        category1.setName("c-aa");
        Category category2 = new Category();
        category2.setName("c-bb");

        Item item1 = new Item();
        item1.setName("item-aa");
        Item item2 = new Item();
        item2.setName("item-aa");

        category1.getItems().add(item1);
        category1.getItems().add(item2);

        category2.getItems().add(item1);
        category2.getItems().add(item2);

        item1.getCategories().add(category1);
        item1.getCategories().add(category2);

        item2.getCategories().add(category1);
        item2.getCategories().add(category2);

        session.save(category1);
        session.save(category2);
        session.save(item1);
        session.save(item2);
    }

```

```
Hibernate: 
    insert  into atguigu.hb_category (hb_name)  values (?)
Hibernate: 
    insert into atguigu.hb_category (hb_name) values (?)
Hibernate: 
    insert into atguigu.hb_item (hb_name) values (?)
Hibernate: 
    insert into atguigu.hb_item (hb_name) values (?)
=destroy=
Hibernate: 
    insert into hb_category_item (hb_item_id, hb_category_id) values (?, ?)
Hibernate: 
    insert into hb_category_item (hb_item_id, hb_category_id) values (?, ?)
Hibernate: 
    insert into hb_category_item (hb_item_id, hb_category_id) values (?, ?)
Hibernate: 
    insert into hb_category_item (hb_item_id, hb_category_id) values (?, ?)

Process finished with exit code 0
```

category表格
|#category表格|hb_id|hb_name|
|-|--|---|
|1|1|c-aa|
|2|2|c-bb|

item表格
|#item表格|hb_id|hb_name|
|-|--|---|
|1|1|item-aa|
|2|2|item-aa|
中间表
|#中间表|hb_category_id|hb_item_id|
|-|--|---|
|1|1|1|
|2|1|2|
|3|2|1|
|4|2|2|

