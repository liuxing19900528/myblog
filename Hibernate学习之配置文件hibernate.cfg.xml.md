>hibernate配置文件主要用于数据库连接和hibernate运行时所需的各种属性
>每个hibernate配置文件对应一个configuration对象。
>hibernate配置文件可以有2中格式：hibernate.properties和hibernate.cfg.xml，现在推荐使用xml格式。

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
        <property name="hibernate.connection.url">jdbc:mysql://localhost:3306/test</property>

        <!-- 配置数据库的方言 -->
        <property name="dialect">org.hibernate.dialect.MySQL5Dialect</property>
        <!--<property name="dialect">org.hibernate.dialect.MySQLInnoDBDialect</property>-->

        <!--是否打印sql语句-->
        <property name="show_sql">true</property>

        <!--是否对sql语句格式化-->
        <property name="format_sql">true</property>

        <!--指定自动生成数据库表的策略，有4个值-->
        <property name="hbm2ddl.auto">create</property>

        <property name="connection.isolation">2</property>
        <!--<property name="use_identifier_rollback">true</property>-->

        <!--C3P0配置 -->
        <property name="hibernate.connection.provider_class">
            org.hibernate.service.jdbc.connections.internal.C3P0ConnectionProvider
        </property>
        <property name="hibernate.c3p0.max_size">20</property>
        <property name="hibernate.c3p0.min_size">5</property>
        <property name="hibernate.c3p0.timeout">120</property>
        <property name="automaticTestTable">Test</property>
        <property name="hibernate.c3p0.max_statements">100</property>
        <property name="hibernate.c3p0.idle_test_period">120</property>
        <property name="hibernate.c3p0.acquire_increment">1</property>
        <property name="c3p0.testConnectionOnCheckout">true</property>
        <property name="c3p0.idleConnectionTestPeriod">18000</property>
        <property name="c3p0.maxIdleTime">25000</property>
        <property name="c3p0.idle_test_period">120</property>

        <property name="hibernate.jdbc.fetch_size">100</property>
        <property name="hibernate.jdbc.batch_size">30</property>

        <mapping resource="News.hbm.xml"/>
    </session-factory>
</hibernate-configuration>
```


----------
hibernate 使用c3p0
>首先导入jar包
>加入配置

```
<!--C3P0配置 -->
<property name="hibernate.connection.provider_class">org.hibernate.service.jdbc.connections.internal.C3P0ConnectionProvider</property>
<property name="hibernate.c3p0.max_size">10</property>
<property name="hibernate.c3p0.min_size">5</property>
<property name="hibernate.c3p0.acquire_increment">5</property>
<property name="hibernate.c3p0.timeout">20000</property>
<property name="hibernate.c3p0.idle_test_period">20000</property>
<property name="hibernate.c3p0.max_statements">2</property>

```


----------

```
<property name="hibernate.jdbc.fetch_size">100</property>
<property name="hibernate.jdbc.batch_size">30</property>
```
