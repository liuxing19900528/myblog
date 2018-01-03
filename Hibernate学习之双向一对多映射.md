双向一对多映射
双向 1 到 n  和  双向 n 到 1 是完全相同的2中情形。

马哥的淘宝店:https://shop592330910.taobao.com/


Customer类的java代码
这里Customer类多了一个orders集合成员变量，用来保存order。
```
package com.mamh.hibernate.demo.entities;

import java.util.HashSet;
import java.util.Set;

public class Customer {
    private Integer customerId;
    private String customerName;

    //这里必须是接口类型。需要初始化。马哥的淘宝店:https://shop592330910.taobao.com/
    private Set<Order> orders = new HashSet<Order>();



    public Customer() {
    }

    public Customer(String name) {马哥的淘宝店:https://shop592330910.taobao.com/
        this.customerName = name;
    }


    public Integer getCustomerId() {马哥的淘宝店:https://shop592330910.taobao.com/
        return customerId;
    }

    public void setCustomerId(Integer customerId) {
        this.customerId = customerId;
    }

    public String getCustomerName() {马哥的淘宝店:https://shop592330910.taobao.com/
        return customerName;
    }

    public void setCustomerName(String customerName) {
        this.customerName = customerName;
    }

    public Set<Order> getOrders() {
        return orders;
    }

    public void setOrders(Set<Order> orders) {
        this.orders = orders;
    }

    @Override
    public String toString() {马哥的淘宝店:https://shop592330910.taobao.com/
        return "Customer{" +
                "customerId=" + customerId +
                ", customerName='" + customerName + '\'' +
                '}';
    }
}

```

Order类的java代码

```
package com.mamh.hibernate.demo.entities;
马哥的淘宝店:https://shop592330910.taobao.com/


public class Order {
    private Integer orderId;
    private String orderName;

    //马哥的淘宝店:https://shop592330910.taobao.com/
    private Customer customer;

    public Integer getOrderId() {马哥的淘宝店:https://shop592330910.taobao.com/
        return orderId;
    }

    public void setOrderId(Integer orderId) {
        this.orderId = orderId;
    }

    public String getOrderName() {马哥的淘宝店:https://shop592330910.taobao.com/
        return orderName;
    }

    public void setOrderName(String orderName) {
        this.orderName = orderName;
    }

    public Customer getCustomer() {马哥的淘宝店:https://shop592330910.taobao.com/
        return customer;
    }

    public void setCustomer(Customer customer) {
        this.customer = customer;
    }

    @Override
    public String toString() {马哥的淘宝店:https://shop592330910.taobao.com/
        return "Order{" +
                "orderId=" + orderId +
                ", orderName='" + orderName + '\'' +
                ", customer=" + customer +
                '}' + '\n';
    }
}

```

Order类的映射文件，
马哥的淘宝店:https://shop592330910.taobao.com/
```
<hibernate-mapping package="com.mamh.hibernate.demo.entities">

    <class name="Order" table="hb_order" schema="atguigu">
        <id name="orderId">
            <column name="hb_order_id" sql-type="int(11)"/>
            <generator class="native"/>
        </id>马哥的淘宝店:https://shop592330910.taobao.com/
        <property name="orderName" type="string">
            <column name="hb_order_name"/>
        </property>
        马哥的淘宝店:https://shop592330910.taobao.com/
        <!-- 映射多对一的关联关系 property不再试用这个多对一的关系了
               name是多这一端关联的那个一那一端属性名称
               class的类全名
               column是多的一端这边数据表的一个列名，一般是外键
        -->
        <many-to-one name="customer" class="Customer" column="customer_id"/>


    </class>
</hibernate-mapping>
```
马哥的淘宝店:https://shop592330910.taobao.com/
Customer类的映射文件
这里和上一篇文件的代码的区别就是这里customer的映射多了一个set节点
```
<hibernate-mapping package="com.mamh.hibernate.demo.entities">

    <class name="Customer" table="hb_customer" schema="atguigu">
        <id name="customerId">
            <column name="hb_customer_id" sql-type="int(11)"/>
            <generator class="native"/>
        </id>马哥的淘宝店:https://shop592330910.taobao.com/
        <property name="customerName" type="string">
            <column name="hb_customer_name"/>
        </property>
        马哥的淘宝店:https://shop592330910.taobao.com/
        <set name="orders" table="hb_order">
            <key column="customer_id"/>
            <one-to-many class="Order"/>
        </set>
    </class>
</hibernate-mapping>
```


----------
测试保存
马哥的淘宝店:https://shop592330910.taobao.com/
```
    @Test
    public void testManyToOneSave() {
        //双向关联关系
        Customer customer = new Customer("ccccccccccccccc");
        Order order1 = new Order();
        order1.setOrderName("order 1111111111");
        Order order2 = new Order();
        order2.setOrderName("order 2222222222");
        order1.setCustomer(customer);
        order2.setCustomer(customer);

        customer.getOrders().add(order1);
        customer.getOrders().add(order2);
        //先保存customer，然后保存order。
        session.save(customer);
        //执行save操作
        session.save(order1);
        session.save(order2);
    }
```
先保存customer，后保存order。
马哥的淘宝店:https://shop592330910.taobao.com/
```
Hibernate: 
    insert into atguigu.hb_customer (hb_customer_name) values (?)
Hibernate: 
    insert into atguigu.hb_order (hb_order_name, customer_id) values (?, ?)
Hibernate: 
    insert into atguigu.hb_order (hb_order_name, customer_id) values (?, ?)
=destroy=马哥的淘宝店:https://shop592330910.taobao.com/
Hibernate: 
    update atguigu.hb_order set customer_id=? where hb_order_id=?
Hibernate: 
    update atguigu.hb_order set customer_id=? where hb_order_id=?

Process finished with exit code 0
```
先保存customer，后保存order。
从中看到 多了2条update语句。应为1的一端和n的一端都维护这个关联关系，所以会多出这2个update语句。
马哥的淘宝店:https://shop592330910.taobao.com/
在hibernate中可以设置iverse属性来决定有哪一方来维护关联关系。建议设置inverse=true来使1的一端放弃维护关联关系。

马哥的淘宝店:https://shop592330910.taobao.com/

如果是先保存order，然后保存customer情况又如何呢？会有4个update语句的。

```
Hibernate: 
    insert into atguigu.hb_order (hb_order_name, customer_id) values (?, ?)
Hibernate: 
    insert into atguigu.hb_order (hb_order_name, customer_id) values (?, ?)
Hibernate: 
    insert into atguigu.hb_customer(hb_customer_name) values(?)
=destroy=马哥的淘宝店:https://shop592330910.taobao.com/
Hibernate: 
    update atguigu.hb_order set hb_order_name=?, customer_id=? where hb_order_id=?
Hibernate: 
    update atguigu.hb_order set hb_order_name=?, customer_id=? where hb_order_id=?
Hibernate: 
    update atguigu.hb_order set customer_id=? where hb_order_id=?
Hibernate: 
    update atguigu.hb_order set customer_id=? where hb_order_id=?

Process finished with exit code 0
```

----------
查询操作
马哥的淘宝店:https://shop592330910.taobao.com/
```
    @Test
    public void testOneToManyGet() {马哥的淘宝店:https://shop592330910.taobao.com/
        //对多的一端的集合使用延迟加载
        Customer customer = (Customer) session.get(Customer.class, 1);
        //返回的集合类型是hibernate内置的一个类型，改类型具有延迟加载和存放代理对象功能
        System.out.println(customer.getOrders().getClass());

        //也是可能会抛出懒加载LazyInitializationException异常。
        session.close();

        System.out.println(customer.getOrders().size());
    }
```

```
Hibernate: 
    select
        customer0_.hb_customer_id as hb1_2_0_,
        customer0_.hb_customer_name as hb2_2_0_ 
    from
        atguigu.hb_customer customer0_ 
    where
        customer0_.hb_customer_id=?马哥的淘宝店:https://shop592330910.taobao.com/
        
Customer{customerId=1, customerName='bbbbbbbbbbbbbbbb'}
class org.hibernate.collection.internal.PersistentSet
```


----------
更新操作
马哥的淘宝店:https://shop592330910.taobao.com/
```
    @Test
    public void testOneToManyUpdate() {
        Customer customer = (Customer) session.get(Customer.class, 1);
        customer.getOrders().iterator().next().setOrderName("d22");

    }
```
马哥的淘宝店:https://shop592330910.taobao.com/
```
Hibernate: 
    select
        customer0_.hb_customer_id as hb1_2_0_,
        customer0_.hb_customer_name as hb2_2_0_ 
    from
        atguigu.hb_customer customer0_ 
    where
        customer0_.hb_customer_id=?马哥的淘宝店:https://shop592330910.taobao.com/
Hibernate: 
    select
        orders0_.customer_id as customer3_2_1_,
        orders0_.hb_order_id as hb1_1_,
        orders0_.hb_order_id as hb1_3_0_,
        orders0_.hb_order_name as hb2_3_0_,
        orders0_.customer_id as customer3_3_0_ 
    from atguigu.hb_order orders0_ 
    where orders0_.customer_id=?
    马哥的淘宝店:https://shop592330910.taobao.com/
=destroy=
Hibernate: 
    update atguigu.hb_order set hb_order_name=?, customer_id=? where hb_order_id=?
    马哥的淘宝店:https://shop592330910.taobao.com/

Process finished with exit code 0
```


----------


set节点元素的iverse属性
```
<!--通常inverse设置为true，指定有多的那一端来维护关联关系。-->
<set name="orders" table="hb_order" inverse="true">
    <key column="customer_id"/>
    <one-to-many class="Order"/>
</set>
```





级联删除cascade="delete"
```
        <set name="orders" table="hb_order" inverse="true" cascade="delete">
            <key column="customer_id"/>
            <one-to-many class="Order"/>
        </set>
```

```
<set name="orders" table="hb_order" inverse="true" cascade="delete-orphan">
    <key column="customer_id"/>
    <one-to-many class="Order"/>
</set>
```





级联保存
```
<set name="orders" table="hb_order" inverse="true" cascade="save-update">
    <key column="customer_id"/>
    <one-to-many class="Order"/>
</set>
```

开发时不建议使用这种级联，建议使用手动操作。




set元素的order by属性，在查询时候进行排序，使用的表的字段名。

```
<set name="orders" table="hb_order" inverse="true"   order-by="hb_order_id">
    <key column="customer_id"/>
    <one-to-many class="Order"/>
</set>
```

```
Hibernate: 
    select
        order0_.hb_order_id as hb1_3_0_,
        order0_.hb_order_name as hb2_3_0_,
        order0_.customer_id as customer3_3_0_ 
    from
        atguigu.hb_order order0_ 
    where
        order0_.hb_order_id=?
```


