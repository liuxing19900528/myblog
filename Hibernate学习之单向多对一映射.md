
一个customer对应多个order。在order中有一个customer的引用。
 https://shop592330910.taobao.com/


Customer类的java代码
```
package com.mamh.hibernate.demo.entities;

public class Customer {
    private Integer customerId;
    private String customerName;

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

    public String getCustomerName() {  https://shop592330910.taobao.com/

        return customerName;
    }

    public void setCustomerName(String customerName) {
        this.customerName = customerName;
    }
}

```
Order类的java代码
```
package com.mamh.hibernate.demo.entities;

public class Order {
    private Integer orderId;
    private String orderName;
                       https://shop592330910.taobao.com/


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
}

```
                      https://shop592330910.taobao.com/


Order类的映射文件，
```
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

                         https://shop592330910.taobao.com/

    </class>
</hibernate-mapping>
```

Customer类的映射文件

```
<hibernate-mapping>

    <class name="com.mamh.hibernate.demo.entities.Customer" table="hb_customer" schema="mage">
        <id name="customerId">
            <column name="hb_customer_id" sql-type="int(11)"/>
            <generator class="native"/>
        </id>
        <property name="customerName" type="string">
            <column name="hb_customer_name"/>
        </property>
    </class>
</hibernate-mapping>
```
                      https://shop592330910.taobao.com/


----------


1.测试保存order和customer的方法

先保存customer，然后保存order。
```
    @Test
    public void testManyToOneSave(){

        Customer customer = new Customer("aa");

        Order order1 = new Order();
        order1.setOrderName("order 1");
        Order order2 = new Order();
        order2.setOrderName("order 2");

        order1.setCustomer(customer);
        order2.setCustomer(customer);

        //执行save操作    https://shop592330910.taobao.com/


        //先保存customer，然后保存order。
        session.save(customer);

        session.save(order1);
        session.save(order2);
    }
```
先保存customer，然后保存order，这个会出现3条插入sql语句。这个是建议这样的顺序的。

```
Hibernate: 
    insert 
    into
        mage.hb_customer
        (hb_customer_name) 
    values
        (?)
Hibernate: 
    insert 
    into
        mage.hb_order
        (hb_order_name, customer_id) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        mage.hb_order
        (hb_order_name, customer_id) 
    values
        (?, ?)
```
如果是先保存order，然后再保存customer，情况由如何呢？

```
//执行save操作       https://shop592330910.taobao.com/

//先保存order，然后保存customer。
session.save(order1);
session.save(order2);

session.save(customer);


```
如果是先保存order，然后再保存customer，能成功的。会出现3条insert语句，最后还有2条update语句。
这中情况就多了2条update语句。因为order表需要一个外键，先保存oder的是还不知道这个外键
customer是多少。后来保存好了customer，就知道customer ID了。最后会更新一些oder中的外键。
这样就多了2条update更新语句。这样多了2条语句效率就会低一些。
```
Hibernate: 
    insert 
    into
        mage.hb_order
        (hb_order_name, customer_id) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        mage.hb_order
        (hb_order_name, customer_id) 
    values
        (?, ?)
Hibernate: 
    insert 
    into
        mage.hb_customer
        (hb_customer_name) 
    values
        (?)
=destroy=
Hibernate: 
    update
        mage.hb_order 
    set
        hb_order_name=?,
        customer_id=? 
    where
        hb_order_id=?
Hibernate: 
    update
        mage.hb_order 
    set
        hb_order_name=?,
        customer_id=? 
    where
        hb_order_id=?
```


----------


2.测试查询order和customer的方法

查询多的一端的一个对象，默认情况下，只查询了多的一端的对象，而没有查询关联的那一端的对象。
就是说查询Order对象，只得到了Order对象，而没有得到Customer对象。
```
    @Test
    public void testManyToOneGet(){   https://shop592330910.taobao.com/

        Order order = (Order) session.get(Order.class, 1);
        System.out.println(order.getOrderName() + ":  "+order.getOrderId());
        //这里值用到了order对象的属性，没有用的customer的属性，所以不会发送取查询customer相关的sql语句的。
    }
```
```
Hibernate: //通过这个hibernate的sql输出我们看到没有去查询customer相关的。
    select
        order0_.hb_order_id as hb1_3_0_,
        order0_.hb_order_name as hb2_3_0_,
        order0_.customer_id as customer3_3_0_ 
    from
        mage.hb_order order0_ 
    where
        order0_.hb_order_id=?
        
order 1:  1
```

如果我们用到了customer的相关属性，例如获取order中的customer对象，
或者直接打印order对象 都是会发送查询customer表的sql语句的。
```
    @Test
    public void testManyToOneGet(){
        Order order = (Order) session.get(Order.class, 1);
        System.out.println(order);

        Customer customer = order.getCustomer();
        System.out.println(customer);
    }
```

```
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

System.out.println(order);的输出结果：      
Order{
    orderId=1, 
    orderName='order 1', 
    customer=Customer{
        customerId=1, 
        customerName='aa'
        }
}

System.out.println(customer);的输出结果：
Customer{
    customerId=1, 
    customerName='aa'
}
```

如果在查询customer对象的时候，有多的一端导航到一的一端时，若此时session被关闭，则
会发生一个LazyInitializationException异常
```
    @Test
    public void testManyToOneGet(){ https://shop592330910.taobao.com/

        Order order = (Order) session.get(Order.class, 1);
        System.out.println(order.getOrderName() + ":  "+order.getOrderId());

        session.close();

        Customer customer = order.getCustomer();
        System.out.println(customer);
    }
```

```
Hibernate: 
    select
        order0_.hb_order_id as hb1_3_0_,
        order0_.hb_order_name as hb2_3_0_,
        order0_.customer_id as customer3_3_0_ 
    from
        mage.hb_order order0_ 
    where
        order0_.hb_order_id=?
order 1:  1
=destroy=

org.hibernate.LazyInitializationException: could not initialize proxy - no Session

    at org.hibernate.proxy.AbstractLazyInitializer.initialize(AbstractLazyInitializer.java:149)
    at org.hibernate.proxy.AbstractLazyInitializer.getImplementation(AbstractLazyInitializer.java:195)
    at org.hibernate.proxy.pojo.javassist.JavassistLazyInitializer.invoke(JavassistLazyInitializer.java:185)
    at com.mamh.hibernate.demo.entities.Customer_$$_javassist_1.toString(Customer_$$_javassist_1.java)
    at java.lang.String.valueOf(String.java:2994)
    at java.io.PrintStream.println(PrintStream.java:821)
    at com.mamh.hibernate.demo.HibernateTest.testManyToOneGet(HibernateTest.java:61)
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
    at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    at java.lang.reflect.Method.invoke(Method.java:498)
    at org.junit.runners.model.FrameworkMethod$1.runReflectiveCall(FrameworkMethod.java:50)
    at org.junit.internal.runners.model.ReflectiveCallable.run(ReflectiveCallable.java:12)
    at org.junit.runners.model.FrameworkMethod.invokeExplosively(FrameworkMethod.java:47)
    at org.junit.internal.runners.statements.InvokeMethod.evaluate(InvokeMethod.java:17)
    at org.junit.internal.runners.statements.RunBefores.evaluate(RunBefores.java:26)
    at org.junit.internal.runners.statements.RunAfters.evaluate(RunAfters.java:27)
    at org.junit.runners.ParentRunner.runLeaf(ParentRunner.java:325)
    at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:78)
    at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:57)
    at org.junit.runners.ParentRunner$3.run(ParentRunner.java:290)
    at org.junit.runners.ParentRunner$1.schedule(ParentRunner.java:71)
    at org.junit.runners.ParentRunner.runChildren(ParentRunner.java:288)
    at org.junit.runners.ParentRunner.access$000(ParentRunner.java:58)
    at org.junit.runners.ParentRunner$2.evaluate(ParentRunner.java:268)
    at org.junit.runners.ParentRunner.run(ParentRunner.java:363)
    at org.junit.runner.JUnitCore.run(JUnitCore.java:137)
    at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:68)
    at com.intellij.rt.execution.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:47)
    at com.intellij.rt.execution.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:242)
    at com.intellij.rt.execution.junit.JUnitStarter.main(JUnitStarter.java:70)


```


获取order对象时，默认情况下，其关联的customer对象是一个代理对象！


----------
3.测试更新order，customer对象

```
    @Test
    public void testManyToOneUpdate() {
        Order order = (Order) session.get(Order.class, 1);
        order.getCustomer().setCustomerName("new customer name");
    }
```

```
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
=destroy=
Hibernate: 
    update
        mage.hb_customer 
    set
        hb_customer_name=? 
    where
        hb_customer_id=?
```


----------
4.测试删除order和customer


在不设定级联关系的情况下，且1这一端对象有n的对象在引用，不能直接删除。
也就是说customer 的id有在order中作为外键就不能删除这个customer。
要删除也是先把order中的删除完，然后在删除customer中的。
```
    @Test
    public void testManyToOneDelete() {
        Customer customer = (Customer) session.get(Customer.class, 1);
        session.delete(customer);
    }
```

```
Hibernate: 
    select
        customer0_.hb_customer_id as hb1_2_0_,
        customer0_.hb_customer_name as hb2_2_0_ 
    from
        mage.hb_customer customer0_ 
    where
        customer0_.hb_customer_id=?
=destroy=
Nov 17, 2017 2:51:23 PM org.hibernate.engine.jdbc.spi.SqlExceptionHelper logExceptions
WARN: SQL Error: 1451, SQLState: 23000
Hibernate: 
    delete 
    from
        mage.hb_customer 
    where
        hb_customer_id=?
Nov 17, 2017 2:51:23 PM org.hibernate.engine.jdbc.spi.SqlExceptionHelper logExceptions
ERROR: Cannot delete or update a parent row: a foreign key constraint fails (`mage`.`hb_order`, CONSTRAINT `FK1C0AD3C966B2F7AA` FOREIGN KEY (`customer_id`) REFERENCES `hb_customer` (`hb_customer_id`))
Nov 17, 2017 2:51:23 PM org.hibernate.engine.jdbc.batch.internal.BatchingBatch performExecution
ERROR: HHH000315: Exception executing batch [Cannot delete or update a parent row: a foreign key constraint fails (`mage`.`hb_order`, CONSTRAINT `FK1C0AD3C966B2F7AA` FOREIGN KEY (`customer_id`) REFERENCES `hb_customer` (`hb_customer_id`))]

org.hibernate.exception.ConstraintViolationException: Cannot delete or update a parent row: a foreign key constraint fails (`mage`.`hb_order`, CONSTRAINT `FK1C0AD3C966B2F7AA` FOREIGN KEY (`customer_id`) REFERENCES `hb_customer` (`hb_customer_id`))

    at org.hibernate.exception.internal.SQLStateConversionDelegate.convert(SQLStateConversionDelegate.java:128)
    at org.hibernate.exception.internal.StandardSQLExceptionConverter.convert(StandardSQLExceptionConverter.java:49)
    at org.hibernate.engine.jdbc.spi.SqlExceptionHelper.convert(SqlExceptionHelper.java:125)
    at org.hibernate.engine.jdbc.spi.SqlExceptionHelper.convert(SqlExceptionHelper.java:110)
    at org.hibernate.engine.jdbc.internal.proxy.AbstractStatementProxyHandler.continueInvocation(AbstractStatementProxyHandler.java:129)
    at org.hibernate.engine.jdbc.internal.proxy.AbstractProxyHandler.invoke(AbstractProxyHandler.java:81)
    at com.sun.proxy.$Proxy9.executeBatch(Unknown Source)
    at org.hibernate.engine.jdbc.batch.internal.BatchingBatch.performExecution(BatchingBatch.java:110)
    at org.hibernate.engine.jdbc.batch.internal.BatchingBatch.doExecuteBatch(BatchingBatch.java:101)
    at org.hibernate.engine.jdbc.batch.internal.AbstractBatchImpl.execute(AbstractBatchImpl.java:161)
    at org.hibernate.engine.jdbc.internal.JdbcCoordinatorImpl.executeBatch(JdbcCoordinatorImpl.java:162)
    at org.hibernate.engine.spi.ActionQueue.executeActions(ActionQueue.java:357)
    at org.hibernate.engine.spi.ActionQueue.executeActions(ActionQueue.java:280)
    at org.hibernate.event.internal.AbstractFlushingEventListener.performExecutions(AbstractFlushingEventListener.java:326)
    at org.hibernate.event.internal.DefaultFlushEventListener.onFlush(DefaultFlushEventListener.java:52)
    at org.hibernate.internal.SessionImpl.flush(SessionImpl.java:1127)
    at org.hibernate.internal.SessionImpl.managedFlush(SessionImpl.java:325)
    at org.hibernate.engine.transaction.internal.jdbc.JdbcTransaction.beforeTransactionCommit(JdbcTransaction.java:101)
    at org.hibernate.engine.transaction.spi.AbstractTransactionImpl.commit(AbstractTransactionImpl.java:175)
    at com.mamh.hibernate.demo.HibernateTest.destroy(HibernateTest.java:315)
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
    at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    at java.lang.reflect.Method.invoke(Method.java:498)
    at org.junit.runners.model.FrameworkMethod$1.runReflectiveCall(FrameworkMethod.java:50)
    at org.junit.internal.runners.model.ReflectiveCallable.run(ReflectiveCallable.java:12)
    at org.junit.runners.model.FrameworkMethod.invokeExplosively(FrameworkMethod.java:47)
    at org.junit.internal.runners.statements.RunAfters.evaluate(RunAfters.java:33)
    at org.junit.runners.ParentRunner.runLeaf(ParentRunner.java:325)
    at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:78)
    at org.junit.runners.BlockJUnit4ClassRunner.runChild(BlockJUnit4ClassRunner.java:57)
    at org.junit.runners.ParentRunner$3.run(ParentRunner.java:290)
    at org.junit.runners.ParentRunner$1.schedule(ParentRunner.java:71)
    at org.junit.runners.ParentRunner.runChildren(ParentRunner.java:288)
    at org.junit.runners.ParentRunner.access$000(ParentRunner.java:58)
    at org.junit.runners.ParentRunner$2.evaluate(ParentRunner.java:268)
    at org.junit.runners.ParentRunner.run(ParentRunner.java:363)
    at org.junit.runner.JUnitCore.run(JUnitCore.java:137)
    at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:68)
    at com.intellij.rt.execution.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:47)
    at com.intellij.rt.execution.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:242)
    at com.intellij.rt.execution.junit.JUnitStarter.main(JUnitStarter.java:70)
Caused by: java.sql.BatchUpdateException: Cannot delete or update a parent row: a foreign key constraint fails (`mage`.`hb_order`, CONSTRAINT `FK1C0AD3C966B2F7AA` FOREIGN KEY (`customer_id`) REFERENCES `hb_customer` (`hb_customer_id`))
    at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
    at sun.reflect.NativeConstructorAccessorImpl.newInstance(NativeConstructorAccessorImpl.java:62)
    at sun.reflect.DelegatingConstructorAccessorImpl.newInstance(DelegatingConstructorAccessorImpl.java:45)
    at java.lang.reflect.Constructor.newInstance(Constructor.java:423)
    at com.mysql.cj.core.util.Util.handleNewInstance(Util.java:185)
    at com.mysql.cj.core.util.Util.getInstance(Util.java:168)
    at com.mysql.cj.core.util.Util.getInstance(Util.java:175)
    at com.mysql.cj.jdbc.exceptions.SQLError.createBatchUpdateException(SQLError.java:636)
    at com.mysql.cj.jdbc.PreparedStatement.executeBatchSerially(PreparedStatement.java:1734)
    at com.mysql.cj.jdbc.PreparedStatement.executeBatchInternal(PreparedStatement.java:1217)
    at com.mysql.cj.jdbc.StatementImpl.executeBatch(StatementImpl.java:1009)
    at com.mchange.v2.c3p0.impl.NewProxyPreparedStatement.executeBatch(NewProxyPreparedStatement.java:1723)
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
    at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    at java.lang.reflect.Method.invoke(Method.java:498)
    at org.hibernate.engine.jdbc.internal.proxy.AbstractStatementProxyHandler.continueInvocation(AbstractStatementProxyHandler.java:122)
    ... 37 more
Caused by: java.sql.SQLIntegrityConstraintViolationException: Cannot delete or update a parent row: a foreign key constraint fails (`mage`.`hb_order`, CONSTRAINT `FK1C0AD3C966B2F7AA` FOREIGN KEY (`customer_id`) REFERENCES `hb_customer` (`hb_customer_id`))
    at com.mysql.cj.jdbc.exceptions.SQLError.createSQLException(SQLError.java:533)
    at com.mysql.cj.jdbc.exceptions.SQLError.createSQLException(SQLError.java:513)
    at com.mysql.cj.jdbc.exceptions.SQLExceptionsMapping.translateException(SQLExceptionsMapping.java:115)
    at com.mysql.cj.jdbc.ConnectionImpl.execSQL(ConnectionImpl.java:1983)
    at com.mysql.cj.jdbc.PreparedStatement.executeInternal(PreparedStatement.java:1826)
    at com.mysql.cj.jdbc.PreparedStatement.executeUpdateInternal(PreparedStatement.java:2034)
    at com.mysql.cj.jdbc.PreparedStatement.executeBatchSerially(PreparedStatement.java:1712)
    ... 45 more


Process finished with exit code 255
```
