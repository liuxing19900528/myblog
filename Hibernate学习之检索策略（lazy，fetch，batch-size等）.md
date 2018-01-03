https://github.com/mmh891113/myblog.git


马哥的淘宝店:https://shop592330910.taobao.com/

Hibernate学习之检索策略

马哥的淘宝店:https://shop592330910.taobao.com/
检索数据时的2个问题
不浪费内存：当hibernate从数据库中加载customer对象时，如果同时加载所有关联的order对象，
而程序实际上仅仅需要访问customer对象，那么这些关联的order对象就白白浪费许多内存。

更高的查询效率：发送尽可能少的sql语句。
马哥的淘宝店:https://shop592330910.taobao.com/

类级别的检索策略

类级别的检索策略包括立即检索和延迟检索，默认是延迟检索。
立即检索：立即加载检索方法指定的对象。
延迟检索：延迟加载检索方法指定的对象，在使用具体的属性时，再进行加载。

类级别的检索策略可以通过class元素的lazy属性进行设置，设置为true或者false。
这2个策略何时使用呢？如果程序加载一个对象的目的是为了访问它的属性，可以采用立即检索。
如果程序加载一个持久化对象的目的仅仅为了获取他的引用，可以采取延迟加载。

注意：延迟加载有可能出现懒加载异常。


```
package com.mamh.hibernate.demo.entities;

import java.util.HashSet;
import java.util.Set;

public class Customer {
    private Integer customerId;
    private String customerName;
马哥的淘宝店:https://shop592330910.taobao.com/
    private Set<Order> orders = new HashSet<Order>();

    public Customer() {
    }

    public Customer(String name) {
        this.customerName = name;
    }


    public Integer getCustomerId() {
        return customerId;
    }

    public void setCustomerId(Integer customerId) {
        this.customerId = customerId;
    }

    public String getCustomerName() {
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
    public String toString() {
        return "Customer{" +
                "customerId=" + customerId +
                ", customerName='" + customerName + '\'' +
                '}';
    }
}

package com.mamh.hibernate.demo.entities;

public class Order {
    private Integer orderId;
    private String orderName;

马哥的淘宝店:https://shop592330910.taobao.com/
    private Customer customer;

    public Integer getOrderId() {
        return orderId;
    }

    public void setOrderId(Integer orderId) {
        this.orderId = orderId;
    }

    public String getOrderName() {
        return orderName;
    }

    public void setOrderName(String orderName) {
        this.orderName = orderName;
    }

    public Customer getCustomer() {
        return customer;
    }

    public void setCustomer(Customer customer) {
        this.customer = customer;
    }

    @Override
    public String toString() {
        return "Order{" +
                "orderId=" + orderId +
                ", orderName='" + orderName + '\'' +
                ", customer=" + customer +
                '}' + '\n';
    }
}



<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-mapping PUBLIC
    "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
    "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping package="com.mamh.hibernate.demo.entities">

    <class name="Order" table="hb_order" schema="mage">
        <id name="orderId">
            <column name="hb_order_id" sql-type="int(11)"/>
            <generator class="native"/>
        </id>
        <property name="orderName" type="string">
            <column name="hb_order_name"/>
        </property>

        <!-- 映射多对一的关联关系 property不再试用这个多对一的关系了
               name是多这一端关联的那个一那一端属性名称
               class的类全名
               column是多的一端这边数据表的一个列名，一般是外键
        -->
        <many-to-one name="customer" class="Customer" column="customer_id"/>


    </class>
</hibernate-mapping>




```
测试类级别的检索策略

```
    @Test马哥的淘宝店:https://shop592330910.taobao.com/
    public void testClassLevelStrategy(){
        //类级别的检索策略

        Customer customer = (Customer) session.load(Customer.class, 1);
        System.out.println(customer.getClass());

    }
```


```
延迟加载lazy="true"
<hibernate-mapping package="com.mamh.hibernate.demo.entities">

    <class name="Customer" table="hb_customer" schema="mage" lazy="true">
        <id name="customerId">
            <column name="hb_customer_id" sql-type="int(11)"/>
            <generator class="native"/>
        </id>
        <property name="customerName" type="string">
            <column name="hb_customer_name"/>
        </property>

        <set name="orders" table="hb_order" inverse="true" cascade="save-update" order-by="hb_order_id">
            <key column="customer_id"/>
            <one-to-many class="Order"/>
        </set>
    </class>
</hibernate-mapping>

输出：
class com.mamh.hibernate.demo.entities.Customer_$$_javassist_1
```


```
立即加载lazy="false"
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping package="com.mamh.hibernate.demo.entities">

    <class name="Customer" table="hb_customer" schema="mage" lazy="false">
        <id name="customerId">
            <column name="hb_customer_id" sql-type="int(11)"/>
            <generator class="native"/>
        </id>
        <property name="customerName" type="string">
            <column name="hb_customer_name"/>
        </property>

        <set name="orders" table="hb_order" inverse="true" cascade="save-update" order-by="hb_order_id">
            <key column="customer_id"/>
            <one-to-many class="Order"/>
        </set>
    </class>
</hibernate-mapping>

输出：
Hibernate: 马哥的淘宝店:https://shop592330910.taobao.com/
    select
        customer0_.hb_customer_id as hb1_2_0_,
        customer0_.hb_customer_name as hb2_2_0_ 
    from
        mage.hb_customer customer0_ 
    where
        customer0_.hb_customer_id=?
class com.mamh.hibernate.demo.entities.Customer
=destroy=
```

一对多和多对多的检索策略

1对多 或 多对多 的集合属性默认是使用懒加载检索策略的。
可以通过设置set元素的lazy属性来重新设置检索策略。默认是true，并不建议设置为false。
set中的lazy还可以取extra，增强型的延迟加载。取该值会尽可能的延迟集合的初始化。



```
    @Test马哥的淘宝店:https://shop592330910.taobao.com/
    public void testOne2ManyLevelStrategy() {
        Customer customer = (Customer) session.get(Customer.class, 1);
        System.out.println(customer.getCustomerName());

        //1. 1对多 或  多对多 的集合属性默认是使用懒加载检索策略的。
        //2. 可以通过设置set元素的lazy属性来重新设置检索策略。

        //System.out.println(customer.getOrders());

    }
```


```
延迟加载：
如果不访问任何order对象是不会取加载它的。
Hibernate: 马哥的淘宝店:https://shop592330910.taobao.com/
    select
        customer0_.hb_customer_id as hb1_2_0_,
        customer0_.hb_customer_name as hb2_2_0_ 
    from
        mage.hb_customer customer0_ 
    where
        customer0_.hb_customer_id=?
aa

Process finished with exit code 0
```


```
延迟加载，
如果 有这样的 调用 customer中的集合order属性的话就会加载oder对象。
System.out.println(customer.getOrders());
Hibernate: 马哥的淘宝店:https://shop592330910.taobao.com/
    select
        customer0_.hb_customer_id as hb1_2_0_,
        customer0_.hb_customer_name as hb2_2_0_ 
    from
        mage.hb_customer customer0_ 
    where
        customer0_.hb_customer_id=?
aa
Hibernate: 马哥的淘宝店:https://shop592330910.taobao.com/
    select
        orders0_.customer_id as customer3_2_1_,
        orders0_.hb_order_id as hb1_1_,
        orders0_.hb_order_id as hb1_3_0_,
        orders0_.hb_order_name as hb2_3_0_,
        orders0_.customer_id as customer3_3_0_ 
    from
        mage.hb_order orders0_ 
    where
        orders0_.customer_id=? 
    order by
        orders0_.hb_order_id
[Order{orderId=1, orderName='aa', customer=Customer{customerId=1, customerName='aa'}}
, Order{orderId=2, orderName='bb', customer=Customer{customerId=1, customerName='aa'}}
, Order{orderId=3, orderName='cc', customer=Customer{customerId=1, customerName='aa'}}
]

Process finished with exit code 0
```



```
可以通过设置customer.hbm.xml对应的set元素的lazy属性来更改这个。
这里lazy="false"就表示立即加载了。

        <set name="orders" table="hb_order" inverse="true" cascade="save-update" order-by="hb_order_id" lazy="false">
            <key column="customer_id"/>
            <one-to-many class="Order"/>
        </set>
```


```
    @Test
    public void testOne2ManyLevelStrategy() {
        Customer customer = (Customer) session.get(Customer.class, 1);
        System.out.println(customer.getCustomerName());
    } 

这里lazy="false"就表示立即加载了。     
Hibernate: 马哥的淘宝店:https://shop592330910.taobao.com/
    select
        customer0_.hb_customer_id as hb1_2_0_,
        customer0_.hb_customer_name as hb2_2_0_ 
    from
        mage.hb_customer customer0_ 
    where
        customer0_.hb_customer_id=?
Hibernate: 马哥的淘宝店:https://shop592330910.taobao.com/
    select
        orders0_.customer_id as customer3_2_1_,
        orders0_.hb_order_id as hb1_1_,
        orders0_.hb_order_id as hb1_3_0_,
        orders0_.hb_order_name as hb2_3_0_,
        orders0_.customer_id as customer3_3_0_ 
    from
        mage.hb_order orders0_ 
    where
        orders0_.customer_id=? 
    order by
        orders0_.hb_order_id
aa

Process finished with exit code 0
```


```
这里lazy="extra"增强型延迟加载。     
    @Test
    public void testOne2ManyLevelStrategy() {马哥的淘宝店:https://shop592330910.taobao.com/
        Customer customer = (Customer) session.get(Customer.class, 1);
        System.out.println(customer.getCustomerName());

        //1. 1对多 或  多对多 的集合属性默认是使用懒加载检索策略的。
        //2. 可以通过设置set元素的lazy属性来重新设置检索策略。


        System.out.println(customer.getOrders().size());

    }
        <set name="orders" table="hb_order" inverse="true" cascade="save-update" order-by="hb_order_id" lazy="extra">
            <key column="customer_id"/>
            <one-to-many class="Order"/>
        </set>


这里访问了order集合的size，但是并没去初始化整个order集合，而是采用了sql语句来获取了集合的个数。

Hibernate: 马哥的淘宝店:https://shop592330910.taobao.com/
    select
        customer0_.hb_customer_id as hb1_2_0_,
        customer0_.hb_customer_name as hb2_2_0_ 
    from
        mage.hb_customer customer0_ 
    where
        customer0_.hb_customer_id=?
aa
Hibernate: 
    select count(hb_order_id) from mage.hb_order  where  customer_id =?
3
```


直接手动调用初始化集合对象


```
        Hibernate.initialize(customer.getOrders());
```
大部分取默认值，lazy= “true”


set元素的batch-size属性

这个可以设定一次初始化集合的数量。



```
    @Test
    public void testBatchSize(){马哥的淘宝店:https://shop592330910.taobao.com/
        List<Customer> customers = session.createQuery("from Customer ").list();

        System.out.println(customers.size());

    }


Hibernate: 马哥的淘宝店:https://shop592330910.taobao.com/
    select
        customer0_.hb_customer_id as hb1_2_,
        customer0_.hb_customer_name as hb2_2_ 
    from
        mage.hb_customer customer0_
4
```



```
   @Test
    public void testBatchSize() {马哥的淘宝店:https://shop592330910.taobao.com/
        List<Customer> customers = session.createQuery("from Customer ").list();

        System.out.println(customers.size());
        for (Customer customer : customers) {
            if (customer.getOrders() != null) {
                System.out.println(customer.getOrders().size());
            }
        }

    }

Hibernate: 马哥的淘宝店:https://shop592330910.taobao.com/
    select
        customer0_.hb_customer_id as hb1_2_,
        customer0_.hb_customer_name as hb2_2_ 
    from
        mage.hb_customer customer0_
4

Hibernate: 马哥的淘宝店:https://shop592330910.taobao.com/
    select
        orders0_.customer_id as customer3_2_1_,
        orders0_.hb_order_id as hb1_1_,
        orders0_.hb_order_id as hb1_3_0_,
        orders0_.hb_order_name as hb2_3_0_,
        orders0_.customer_id as customer3_3_0_ 
    from
        mage.hb_order orders0_ 
    where
        orders0_.customer_id=? 
    order by
        orders0_.hb_order_id
3
Hibernate: 马哥的淘宝店:https://shop592330910.taobao.com/
    select
        orders0_.customer_id as customer3_2_1_,
        orders0_.hb_order_id as hb1_1_,
        orders0_.hb_order_id as hb1_3_0_,
        orders0_.hb_order_name as hb2_3_0_,
        orders0_.customer_id as customer3_3_0_ 
    from
        mage.hb_order orders0_ 
    where
        orders0_.customer_id=? 
    order by
        orders0_.hb_order_id
3
Hibernate: 马哥的淘宝店:https://shop592330910.taobao.com/
    select
        orders0_.customer_id as customer3_2_1_,
        orders0_.hb_order_id as hb1_1_,
        orders0_.hb_order_id as hb1_3_0_,
        orders0_.hb_order_name as hb2_3_0_,
        orders0_.customer_id as customer3_3_0_ 
    from
        mage.hb_order orders0_ 
    where
        orders0_.customer_id=? 
    order by
        orders0_.hb_order_id
3

Hibernate: 马哥的淘宝店:https://shop592330910.taobao.com/
    select
        orders0_.customer_id as customer3_2_1_,
        orders0_.hb_order_id as hb1_1_,
        orders0_.hb_order_id as hb1_3_0_,
        orders0_.hb_order_name as hb2_3_0_,
        orders0_.customer_id as customer3_3_0_ 
    from马哥的淘宝店:https://shop592330910.taobao.com/
        mage.hb_order orders0_ 
    where
        orders0_.customer_id=? 
    order by
        orders0_.hb_order_id
0

Process finished with exit code 0
```





```
这里batch-size设置为4， batch-size="4",这样能检索select语句的个数，之前是4条，现在只有一条了。

        <set name="orders" table="hb_order" inverse="true" cascade="save-update" order-by="hb_order_id" lazy="true" batch-size="4">
            <key column="customer_id"/>
            <one-to-many class="Order"/>
        </set>

Hibernate: 马哥的淘宝店:https://shop592330910.taobao.com/
    select
        customer0_.hb_customer_id as hb1_2_,
        customer0_.hb_customer_name as hb2_2_ 
    from
        mage.hb_customer customer0_
4
Hibernate: 马哥的淘宝店:https://shop592330910.taobao.com/
    select
        orders0_.customer_id as customer3_2_1_,
        orders0_.hb_order_id as hb1_1_,
        orders0_.hb_order_id as hb1_3_0_,
        orders0_.hb_order_name as hb2_3_0_,
        orders0_.customer_id as customer3_3_0_ 
    from
        mage.hb_order orders0_ 
    where马哥的淘宝店:https://shop592330910.taobao.com/
        orders0_.customer_id in (
            ?, ?, ?, ?
        ) 
    order by
        orders0_.hb_order_id
3
3
3
0

Process finished with exit code 0

```



```
这里batch-size设置为2,batch-size="2",这样会有2条select语句。这里为什么是2了？应为customer
表数据是4条，4除以batch-size就是等于2的。

        <set name="orders" table="hb_order" inverse="true" cascade="save-update" order-by="hb_order_id" lazy="true" batch-size="2">
            <key column="customer_id"/>
            <one-to-many class="Order"/>
        </set>


Hibernate:马哥的淘宝店:https://shop592330910.taobao.com/ 
    select
        customer0_.hb_customer_id as hb1_2_,
        customer0_.hb_customer_name as hb2_2_ 
    from
        mage.hb_customer customer0_
4
Hibernate: 马哥的淘宝店:https://shop592330910.taobao.com/
    select
        orders0_.customer_id as customer3_2_1_,
        orders0_.hb_order_id as hb1_1_,
        orders0_.hb_order_id as hb1_3_0_,
        orders0_.hb_order_name as hb2_3_0_,
        orders0_.customer_id as customer3_3_0_ 
    from
        mage.hb_order orders0_ 
    where
        orders0_.customer_id in (
            ?, ?
        ) 
    order by
        orders0_.hb_order_id
3
3
Hibernate: 马哥的淘宝店:https://shop592330910.taobao.com/
    select
        orders0_.customer_id as customer3_2_1_,
        orders0_.hb_order_id as hb1_1_,
        orders0_.hb_order_id as hb1_3_0_,
        orders0_.hb_order_name as hb2_3_0_,
        orders0_.customer_id as customer3_3_0_ 
    from
        mage.hb_order orders0_ 
    where
        orders0_.customer_id in ( ?, ? ) 
    order by
        orders0_.hb_order_id
3
0

Process finished with exit code 0
```





set元素的fetch属性，
当fetch取值为select或subselect，决定初始化order集合的查询语句形式，
若取值为join则决定orders集合被初始化的时机。

若把fetch设置为join，lazy属性取值将被忽略。




```
    @Test
    public void testFetch() {马哥的淘宝店:https://shop592330910.taobao.com/
        List<Customer> customers = session.createQuery("from Customer ").list();

        System.out.println(customers.size());
        for (Customer customer : customers) {
            if (customer.getOrders() != null) {
                System.out.println(customer.getOrders().size());
            }
        }

    }
```


首先设置fetch是select，默认是select值。



```
Hibernate: 马哥的淘宝店:https://shop592330910.taobao.com/
    select
        customer0_.hb_customer_id as hb1_2_,
        customer0_.hb_customer_name as hb2_2_ 
    from
        mage.hb_customer customer0_
4
Hibernate: 马哥的淘宝店:https://shop592330910.taobao.com/
    select
        orders0_.customer_id as customer3_2_1_,
        orders0_.hb_order_id as hb1_1_,
        orders0_.hb_order_id as hb1_3_0_,
        orders0_.hb_order_name as hb2_3_0_,
        orders0_.customer_id as customer3_3_0_ 
    from
        mage.hb_order orders0_ 
    where
        orders0_.customer_id in ( ?, ? ) 
    order by
        orders0_.hb_order_id
3
3
Hibernate: 马哥的淘宝店:https://shop592330910.taobao.com/
    select
        orders0_.customer_id as customer3_2_1_,
        orders0_.hb_order_id as hb1_1_,
        orders0_.hb_order_id as hb1_3_0_,
        orders0_.hb_order_name as hb2_3_0_,
        orders0_.customer_id as customer3_3_0_ 
    from
        mage.hb_order orders0_ 
    where
        orders0_.customer_id in ( ?, ? ) 
    order by
        orders0_.hb_order_id
3
0

Process finished with exit code 0
```

这里设置fetch是subselect。通过子查询的方式来初始化所有set集合。
次数lazy有效，但是batch-szie失效。



```

Hibernate: 马哥的淘宝店:https://shop592330910.taobao.com/
    select
        customer0_.hb_customer_id as hb1_2_,
        customer0_.hb_customer_name as hb2_2_ 
    from
        mage.hb_customer customer0_
4
Hibernate: 马哥的淘宝店:https://shop592330910.taobao.com/
    select
        orders0_.customer_id as customer3_2_1_,
        orders0_.hb_order_id as hb1_1_,
        orders0_.hb_order_id as hb1_3_0_,
        orders0_.hb_order_name as hb2_3_0_,
        orders0_.customer_id as customer3_3_0_ 
    from
        mage.hb_order orders0_ 
    where马哥的淘宝店:https://shop592330910.taobao.com/
        orders0_.customer_id in (
            select customer0_.hb_customer_id 
            from mage.hb_customer customer0_
        ) 
    order by
        orders0_.hb_order_id
3
3
3
0
```


这里fetch取值join。
这里采用的是HQL查询。HQL查询忽略fetch=join的取值。


```
Hibernate: 马哥的淘宝店:https://shop592330910.taobao.com/
    select
        customer0_.hb_customer_id as hb1_2_,
        customer0_.hb_customer_name as hb2_2_ 
    from
        mage.hb_customer customer0_
4
Hibernate:马哥的淘宝店:https://shop592330910.taobao.com/ 
    select马哥的淘宝店:https://shop592330910.taobao.com/
        orders0_.customer_id as customer3_2_1_,
        orders0_.hb_order_id as hb1_1_,
        orders0_.hb_order_id as hb1_3_0_,
        orders0_.hb_order_name as hb2_3_0_,
        orders0_.customer_id as customer3_3_0_ 
    from
        mage.hb_order orders0_ 
    where
        orders0_.customer_id in ( ?, ? ) 
    order by
        orders0_.hb_order_id
3
3
Hibernate:马哥的淘宝店:https://shop592330910.taobao.com/ 
    select马哥的淘宝店:https://shop592330910.taobao.com/
        orders0_.customer_id as customer3_2_1_,
        orders0_.hb_order_id as hb1_1_,
        orders0_.hb_order_id as hb1_3_0_,
        orders0_.hb_order_name as hb2_3_0_,
        orders0_.customer_id as customer3_3_0_ 
    from马哥的淘宝店:https://shop592330910.taobao.com/
        mage.hb_order orders0_ 
    where马哥的淘宝店:https://shop592330910.taobao.com/
        orders0_.customer_id in ( ?, ? ) 
    order by
        orders0_.hb_order_id
3
0

Process finished with exit code 0马哥的淘宝店:https://shop592330910.taobao.com/
```


这里值查询一个customer对象，这时值查询customer对象，这个时候order也初始化了，
这个时候lazy属性是没用的。
使用左外链接的方式检索。
HQL查询忽略fetch=join的取值。



```
   @Test
    public void testFetch2() {马哥的淘宝店:https://shop592330910.taobao.com/
        Customer customer = (Customer) session.get(Customer.class, 1);
        System.out.println(customer.getOrders().size());
    }

Hibernate: 马哥的淘宝店:https://shop592330910.taobao.com/
    select
        customer0_.hb_customer_id as hb1_2_1_,
        customer0_.hb_customer_name as hb2_2_1_,
        orders1_.customer_id as customer3_2_3_,
        orders1_.hb_order_id as hb1_3_,
        orders1_.hb_order_id as hb1_3_0_,
        orders1_.hb_order_name as hb2_3_0_,
        orders1_.customer_id as customer3_3_0_ 
    from马哥的淘宝店:https://shop592330910.taobao.com/
        mage.hb_customer customer0_ 
    left outer join
        mage.hb_order orders1_ 
            on customer0_.hb_customer_id=orders1_.customer_id 
    where马哥的淘宝店:https://shop592330910.taobao.com/
        customer0_.hb_customer_id=? 
    order by
        orders1_.hb_order_id
3


```



```
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-mapping PUBLIC
    "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
    "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping package="com.mamh.hibernate.demo.entities">

    <class name="Order" table="hb_order" schema="mage">
        <id name="orderId">
            <column name="hb_order_id" sql-type="int(11)"/>
            <generator class="native"/>
        </id>
        <property name="orderName" type="string">
            <column name="hb_order_name"/>
        </property>

        <!-- 映射多对一的关联关系 property不再试用这个多对一的关系了
               name是多这一端关联的那个一那一端属性名称
               class的类全名
               column是多的一端这边数据表的一个列名，一般是外键
        -->
        <many-to-one name="customer" class="Customer" column="customer_id"/>


    </class>
</hibernate-mapping>
```



```
  @Test
    public void testManyToOneGet() {
        Order order = (Order) session.get(Order.class, 1);
        System.out.println(order.getOrderName() + ":  " + order.getOrderId());

        //Customer customer = order.getCustomer();
        //System.out.println(customer);
    }
    
    
    
    Hibernate: 
    select
        order0_.hb_order_id as hb1_3_0_,
        order0_.hb_order_name as hb2_3_0_,
        order0_.customer_id as customer3_3_0_ 
    from
        mage.hb_order order0_ 
    where
        order0_.hb_order_id=?
aa:  1
```
lazy属性设置为true或者false，表示延迟加载，还是立即加载。当设置了 lazy=”false”


```
        <many-to-one name="customer" class="Customer" column="customer_id"
                     lazy="false"
        />

Hibernate: 
    select
        order0_.hb_order_id as hb1_3_0_,
        order0_.hb_order_name as hb2_3_0_,
        order0_.customer_id as customer3_3_0_ 
    from
        mage.hb_order order0_ 
    where
        order0_.hb_order_id=?
Hibernate: 
    select
        customer0_.hb_customer_id as hb1_2_0_,
        customer0_.hb_customer_name as hb2_2_0_ 
    from
        mage.hb_customer customer0_ 
    where
        customer0_.hb_customer_id=?
aa:  1
```

fetch属性取值为join，表示采用迫切左外链接的方式初始化n关联的1的一端的属性。fetch=”join“



```
Hibernate: 
    select
        order0_.hb_order_id as hb1_3_1_,
        order0_.hb_order_name as hb2_3_1_,
        order0_.customer_id as customer3_3_1_,
        customer1_.hb_customer_id as hb1_2_0_,
        customer1_.hb_customer_name as hb2_2_0_ 
    from
        mage.hb_order order0_ 
    left outer join
        mage.hb_customer customer1_ 
            on order0_.customer_id=customer1_.hb_customer_id 
    where
        order0_.hb_order_id=?
aa:  1
```











































fetch属性取值为join，表示采用迫切左外链接的方式初始化n关联的1的一端的属性。fetch=”join“







