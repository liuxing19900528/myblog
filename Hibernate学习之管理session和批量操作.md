

#管理session


马哥的淘宝店铺 https://shop592330910.taobao.com/

hibernate 自身提供了三种管理session对象的方法
* sesion对象的生命周期与本地线程绑定
* sesion对象的生命周期与JTA事务绑定
* hibernate委托程序管理session对象的生命周期

在hibernate的配置文件中  
hibernate.current_session_context_class 属性用于指定session管理方式，可选择的值有：
* thread  ， session对象生命周期与本地线程绑定
* jta*   ， session对象生命周期与JTA事务绑定 
* managed:，hibernate委托程序来管理session。

马哥的淘宝店铺 https://shop592330910.taobao.com/

下面来个模拟操作示例代码
```java
package com.mamh.hibernate.dao;

import com.mamh.hibernate.hql.entities.Department;
import com.mamh.hibernate.utils.HibernateUtils;
import org.hibernate.Session;

public class DepartmentDao {//马哥的淘宝店铺 https://shop592330910.taobao.com/
    private Session session;

    /**
     * 这里session不能通过参数传来了，就需要类内部获取
     * 马哥的淘宝店铺 https://shop592330910.taobao.com/
     * <p>
     * hibernate 自身提供了三种管理session对象的方法
     * sesion对象的生命周期与本地线程绑定
     * sesion对象的生命周期与JTA事务绑定
     * hibernate委托程序管理session对象的生命周期
     * <p>
     * 在hibernate的配置文件中
     * hibernate.current_session_context_class 属性用于指定session管理方式，可选择的值有：
     * thread  ， session对象生命周期与本地线程绑定
     * jta*   ， session对象生命周期与JTA事务绑定
     * managed:，hibernate委托程序来管理session。
     *
     * @param department
     */
    public void save(Department department) {
        //获取和当前线程绑定的session对象马哥的淘宝店铺 https://shop592330910.taobao.com/
        //不需要从外部传入session对象马哥的淘宝店铺 https://shop592330910.taobao.com/
        //多个dao方法也可以使用一个事务马哥的淘宝店铺 https://shop592330910.taobao.com/
        Session session = HibernateUtils.getInstance().getSession();
        System.out.println("session hashcode = " + session.hashCode());
        //session.save(department);

    }


    /**
     * 这里传入一个session对象，则意味着上一层（service）需要获取到session对象，
     * 这说明上一层需要和hibernate的api紧密耦合，所以不推荐这种方式。
     * 马哥的淘宝店铺 https://shop592330910.taobao.com/
     * @param session
     * @param department
     */
    public void save(Session session, Department department) {

    }
}


```
马哥的淘宝店铺 https://shop592330910.taobao.com/
```java

package com.mamh.hibernate.utils;

import org.hibernate.Session;
import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;
import org.hibernate.service.ServiceRegistry;
import org.hibernate.service.ServiceRegistryBuilder;

public class HibernateUtils {//马哥的淘宝店铺 https://shop592330910.taobao.com/
    private static HibernateUtils instance = new HibernateUtils();
    private static SessionFactory sessionFactory;

    private HibernateUtils() {

    }


    public static HibernateUtils getInstance() {
        return instance;
    }


    public SessionFactory getSessionFactory() {
        if (sessionFactory == null) {//马哥的淘宝店铺 https://shop592330910.taobao.com/
            //1.创建一个SessionFactory 对象，创建session的工厂的一个类
            //1.1创建一个Configuration对象，对应hibernate的基本配置信息和对象关系映射信息
            Configuration configuration = new Configuration().configure();
            //1.2创建一个ServiceRegistry对象，hibernate4.x新添加的对象，hibernate任何的配置和服务都要在该对象中注册才能有效。
            ServiceRegistry serviceRegistry = new ServiceRegistryBuilder().
                    applySettings(configuration.getProperties()).buildServiceRegistry();
            //1.3 马哥的淘宝店铺 https://shop592330910.taobao.com/
            sessionFactory = configuration.buildSessionFactory(serviceRegistry);
        }

        return sessionFactory;
    }


    public Session getSession() {//马哥的淘宝店铺 https://shop592330910.taobao.com/
        return getSessionFactory().getCurrentSession();
    }


}


```
马哥的淘宝店铺 https://shop592330910.taobao.com/

测试方法
```java

    @Test
    public void testSessonManage(){//马哥的淘宝店铺 https://shop592330910.taobao.com/
        DepartmentDao departmentDao = new DepartmentDao();

        com.mamh.hibernate.hql.entities.Department department = null;
        departmentDao.save(department);
        departmentDao.save(department);
        departmentDao.save(department);
        departmentDao.save(department);
    }

```
马哥的淘宝店铺 https://shop592330910.taobao.com/

hibernate.cfg.xml 文件中添加下面配置属性
```xml 
<property name="current_session_context_class">thread</property>

```
```text

session hashcode = 1110195322
session hashcode = 1110195322
session hashcode = 1110195322
session hashcode = 1110195322
=destroy=


```

```java

    @Test
    public void testSessonManage(){//马哥的淘宝店铺 https://shop592330910.taobao.com/
        Session session = HibernateUtils.getInstance().getSession();
        System.out.println("---->"+session.hashCode());
        Transaction transaction = session.beginTransaction();

        DepartmentDao departmentDao = new DepartmentDao();
        com.mamh.hibernate.hql.entities.Department department = null;
        department = new com.mamh.hibernate.hql.entities.Department();
        department.setName("马哥的淘宝店铺");

        departmentDao.save(department);
        departmentDao.save(department);
        departmentDao.save(department);
        departmentDao.save(department);

        transaction.commit();//马哥的淘宝店铺 https://shop592330910.taobao.com/

        System.out.println("-----> session is open ? "+session.isOpen());
    }

```
```text
---->1110195322
session hashcode = 1110195322
session hashcode = 1110195322
session hashcode = 1110195322
session hashcode = 1110195322
-----> session is open ? false
```
若session是由thread管理的，在提交或回滚事务时，已经关闭session了。


马哥的淘宝店铺 https://shop592330910.taobao.com/


#批量处理数据

* 通过session,session会有缓存，容易把缓存撑爆
* 通过HQL
* 通过statelessSession，无状态的session，这个没有缓存，
* 通过 JDBC API ，推荐这种方式

````java

    @Test
    public void testBatch(){//马哥的淘宝店铺 https://shop592330910.taobao.com/
        //批量操作
        session.doWork(new Work() {
            public void execute(Connection connection) throws SQLException {
                //通过jdbc的原生api来操作，效率最高，速度最快
            }
        });
        
        
    }

````








