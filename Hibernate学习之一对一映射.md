Hibernate学习之一对一映射

#基于外键的一对一关联关系

```
package com.mamh.hibernate.demo.entities;
    马哥的淘宝店:https://shop592330910.taobao.com/
public class Department {
    private int depatmentId;
    private String departmentName;

    private Manager manager;

    public Manager getManager() {
        return manager;
    }
    马哥的淘宝店:https://shop592330910.taobao.com/
    public void setManager(Manager manager) {
        this.manager = manager;
    }

    public int getDepatmentId() {
        return depatmentId;
    }

    public void setDepatmentId(int depatmentId) {
        this.depatmentId = depatmentId;
    }

    public String getDepartmentName() {
        return departmentName;
    }
    马哥的淘宝店:https://shop592330910.taobao.com/
    public void setDepartmentName(String departmentName) {
        this.departmentName = departmentName;
    }
}

```

```
package com.mamh.hibernate.demo.entities;
    马哥的淘宝店:https://shop592330910.taobao.com/
public class Manager {
    private int managerId;
    private String managerName;

    private Department department;

    public Department getDepartment() {
        return department;
    }

    public void setDepartment(Department department) {
        this.department = department;
    }

    public int getManagerId() {
        return managerId;
    }

    public void setManagerId(int managerId) {
        this.managerId = managerId;
    }

    public String getManagerName() {
        return managerName;
    }

    public void setManagerName(String managerName) {
        this.managerName = managerName;
    }
}
    马哥的淘宝店:https://shop592330910.taobao.com/
```

```
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping package="com.mamh.hibernate.demo.entities">
    马哥的淘宝店:https://shop592330910.taobao.com/
    <class name="Department" table="hb_department" schema="mage">
        <id name="depatmentId">
            <column name="hb_department_id" sql-type="int(11)"/>
            <generator class="native"/>
        </id>
        <property name="departmentName" type="string">
            <column name="hb_department_name"/>
        </property>

        <!-- 使用many to one 方式来映射一对一关联关系 -->
        <many-to-one name="manager" class="Manager" column="hb_manager_id" unique="true" />

    马哥的淘宝店:https://shop592330910.taobao.com/
    </class>
</hibernate-mapping>
```

马哥的淘宝店:https://shop592330910.taobao.com/
```
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">
<hibernate-mapping package="com.mamh.hibernate.demo.entities">

    <class name="Manager" table="hb_manager" schema="mage">
        <id name="managerId">
            <column name="hb_manager_id" sql-type="int(11)"/>
            <generator class="native"/>
        </id>
        <property name="managerName" type="string">
            <column name="hb_manager_name"/>
        </property>

        <!--映射1to1,关联关系  -->
        <one-to-one name="department" class="Department"/>

    </class>
</hibernate-mapping>
```
马哥的淘宝店:https://shop592330910.taobao.com/
  

----------
保存操作

```

    @Test
    public void testDepartment(){
        Department department = new Department();
        department.setDepartmentName("dept-aaa");

        Manager manager = new Manager();
        manager.setManagerName("mgr-aaa");

        manager.setDepartment(department);
        department.setManager(manager);

        //建议先保存没有外键的那个对象。这样会减少update语句。
        session.save(manager);
        session.save(department);
        马哥的淘宝店:https://shop592330910.taobao.com/

    }
```


----------

查询操作

```
    @Test
    public void testGetDepartment() {
        Department department = (Department) session.get(Department.class, 1);
        System.out.println(department.getDepartmentName());

    }
```
默认情况下会使用懒加载。如果这里不访问manager的话就不会去查询manager对象。
```
Hibernate: 
    select
        department0_.hb_department_id as hb1_4_0_,
        department0_.hb_department_name as hb2_4_0_,
        department0_.hb_manager_id as hb3_4_0_ 
    from
        atguigu.hb_department department0_ 
    where
        department0_.hb_department_id=?
        
dept-aa
```

这里我继续，来查询一些manager对象。
```
    @Test
    public void testGetDepartment() {马哥的淘宝店:https://shop592330910.taobao.com/
        Department department = (Department) session.get(Department.class, 1);
        System.out.println(department.getManager());

    }
```
但是我们发现所使用的sql有点问题
开始查询department是没问题，查manager使用了left outer join左外连接。发现链接条件不对。
查询条件应该是department.manger_Id = manager.manger_id.
```
Hibernate: 马哥的淘宝店:https://shop592330910.taobao.com/
    select
        department0_.hb_department_id as hb1_4_0_,
        department0_.hb_department_name as hb2_4_0_,
        department0_.hb_manager_id as hb3_4_0_ 
    from
        atguigu.hb_department department0_ 
    where
        department0_.hb_department_id=?
Hibernate: 马哥的淘宝店:https://shop592330910.taobao.com/
    select
        manager0_.hb_manager_id as hb1_5_1_,
        manager0_.hb_manager_name as hb2_5_1_,
        department1_.hb_department_id as hb1_4_0_,
        department1_.hb_department_name as hb2_4_0_,
        department1_.hb_manager_id as hb3_4_0_ 
    from
        atguigu.hb_manager manager0_ 
    left outer join
        atguigu.hb_department department1_ 
            on manager0_.hb_manager_id=department1_.hb_department_id 
    where
        manager0_.hb_manager_id=?
com.mamh.hibernate.demo.entities.Manager@64ec96c6
=destroy=马哥的淘宝店:https://shop592330910.taobao.com/
```
正确的做法是：配置一个property-ref属性。

```
<one-to-one name="department" class="Department" property-ref="manager"/>
```
```
Hibernate:马哥的淘宝店:https://shop592330910.taobao.com/ 
    select
        department0_.hb_department_id as hb1_4_0_,
        department0_.hb_department_name as hb2_4_0_,
        department0_.hb_manager_id as hb3_4_0_ 
    from
        atguigu.hb_department department0_ 
    where
        department0_.hb_department_id=?
Hibernate:马哥的淘宝店:https://shop592330910.taobao.com/ 
    select
        manager0_.hb_manager_id as hb1_5_1_,
        manager0_.hb_manager_name as hb2_5_1_,
        department1_.hb_department_id as hb1_4_0_,
        department1_.hb_department_name as hb2_4_0_,
        department1_.hb_manager_id as hb3_4_0_ 
    from
        atguigu.hb_manager manager0_ 
    left outer join
        atguigu.hb_department department1_ 
            on manager0_.hb_manager_id=department1_.hb_manager_id 
    where
        manager0_.hb_manager_id=?
Hibernate:马哥的淘宝店:https://shop592330910.taobao.com/ 
    select
        department0_.hb_department_id as hb1_4_0_,
        department0_.hb_department_name as hb2_4_0_,
        department0_.hb_manager_id as hb3_4_0_ 
    from
        atguigu.hb_department department0_ 
    where
        department0_.hb_manager_id=?
com.mamh.hibernate.demo.entities.Manager@456d6c1e
=destroy=马哥的淘宝店:https://shop592330910.taobao.com/
```
没有外键的这一端，也就是manager这一端，需要设置property-ref来使用主键以外的列作为关联。

我们来测试单独查询manager会是咋样的？
我们发现查询manager的时候，也就是查询没有外键的那个对象会使用左外连接，
一并查出其关联的对象，并一起进行初始化。
```
    @Test
    public void testGetManager() {
        Manager manager = (Manager) session.get(Manager.class, 1);
        System.out.println(manager);
    }马哥的淘宝店:https://shop592330910.taobao.com/
```

```
Hibernate: 马哥的淘宝店:https://shop592330910.taobao.com/
    select
        manager0_.hb_manager_id as hb1_5_1_,
        manager0_.hb_manager_name as hb2_5_1_,
        department1_.hb_department_id as hb1_4_0_,
        department1_.hb_department_name as hb2_4_0_,
        department1_.hb_manager_id as hb3_4_0_ 
    from
        atguigu.hb_manager manager0_ 
    left outer join
        atguigu.hb_department department1_ 
            on manager0_.hb_manager_id=department1_.hb_manager_id 
    where
        manager0_.hb_manager_id=?
        
com.mamh.hibernate.demo.entities.Manager@2ca26d77
=destroy=马哥的淘宝店:https://shop592330910.taobao.com/
```

最后：不要两个表里都使用一个外键来关联另外的一个对象。



----------


#基于主键的一对一关联关系

两张表的主键一样就是基于主键的一对一映射
Department 类和Manager类还是那个类，代码不变

hbm配置映射文件需要改动一下

在department.hbm.xml中使用one to one来映射，同时要使用constrained="true"属性。
```
<hibernate-mapping package="com.mamh.hibernate.demo.entities">

    <class name="Department" table="hb_department" schema="atguigu">
        <id name="depatmentId">
            <column name="hb_department_id" sql-type="int(11)"/>
            <generator class="foreign">
                <param name="property">manager</param>
            </generator>
        </id>
        <property name="departmentName" type="string">
            <column name="hb_department_name"/>
        </property>

        <!-- 使用one to one 方式来映射一对一关联关系 -->
        <!-- 采用foreign主键生成器的一端需要设置constrainend=true -->
        <one-to-one name="manager" class="Manager" constrained="true"/>


    </class>
</hibernate-mapping>
```

```
<hibernate-mapping package="com.mamh.hibernate.demo.entities">

    <class name="Manager" table="hb_manager" schema="atguigu">
        <id name="managerId">
            <column name="hb_manager_id" sql-type="int(11)"/>
            <generator class="native"/>
        </id>
        <property name="managerName" type="string">
            <column name="hb_manager_name"/>
        </property>

        <!--映射1to1,关联关系  -->
        <one-to-one name="department" class="Department" />

    </class>
</hibernate-mapping>
```

```
    @Test
    public void testSaveDepartment() {
        Department department = new Department();
        department.setDepartmentName("dept-aaa");

        Manager manager = new Manager();
        manager.setManagerName("mgr-aaa");

        manager.setDepartment(department);
        department.setManager(manager);

        //先插入manager，然后插入department。这2个调换顺序也是一样的，应为department的主键是有manager决定的，所以要先插入manager。
        session.save(manager);
        session.save(department);


    }
```

```

Hibernate: 
    insert into  atguigu.hb_manager (hb_manager_name)  values (?)
=destroy=
Hibernate: 
    insert into atguigu.hb_department (hb_department_name, hb_department_id) 
    values (?, ?)
```


----------
查询操作

查询department
```
    @Test
    public void testGetDepartment() {
        Department department = (Department) session.get(Department.class, 1);
        System.out.println(department.getDepartmentName());

    }
```

```
Hibernate: 
    select
        department0_.hb_department_id as hb1_4_0_,
        department0_.hb_department_name as hb2_4_0_ 
    from
        atguigu.hb_department department0_ 
    where
        department0_.hb_department_id=?
dept-aaa
```

查询manager
此时会使用一个左外连接来查询，会连带department也查询出来。
```
    @Test
    public void testGetManager() {
        Manager manager = (Manager) session.get(Manager.class, 1);
        System.out.println(manager.getManagerName());
    }
```

```
Hibernate: 
    select
        manager0_.hb_manager_id as hb1_5_1_,
        manager0_.hb_manager_name as hb2_5_1_,
        department1_.hb_department_id as hb1_4_0_,
        department1_.hb_department_name as hb2_4_0_ 
    from
        atguigu.hb_manager manager0_ 
    left outer join
        atguigu.hb_department department1_ 
            on manager0_.hb_manager_id=department1_.hb_department_id 
    where
        manager0_.hb_manager_id=?
mgr-aaa
=destroy=
```
