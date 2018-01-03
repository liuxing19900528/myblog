**马哥的淘宝店:https://shop592330910.taobao.com/**



##Servlet监听器分类

 1. 监听域对象自身的创建和销毁的事件监听器
 2. 监听域对象中的属性的增加和删除的事件监听器
 3. 监听绑定到httpsession域中的某个对象的状态的事件监听器

##监听域对象自身的创建和销毁

实现对应的接口，web.xml中进行注册

###1.ServletContext
        ServletContextListener接口
ServletContextListener
1）监听ServletContext对象被创建或销毁的监听器
2）创建一个实现了ServletContextListener接口的类，并且实现其中的2个方法

```
public class HelloServletContextListener implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent servletContextEvent) {
        System.out.println("ServletContext contextInitialized 创建");
    }

    @Override
    public void contextDestroyed(ServletContextEvent servletContextEvent) {
        System.out.println("ServletContext contextDestroyed 销毁");
    }
}
```
在web.xml文件中配置这个listener

```
    <listener>
        <listener-class>com.马哥私房菜.mvcapp.listener.HelloServletContextListener</listener-class>
    </listener>
```
3）这个是最常用的listener。可以在web应用被加载时对web应用的相关资源进行初始化操作（如创建数据库链接池。创建spring的ioc容器。读取当前web应用的初始化参数）

4）API

```
    //ServletContext对象创建时候（当前web应用被加载时候），servlet容器调用此方法。
    public void contextInitialized(ServletContextEvent servletContextEvent) 

    //ServletContext对象销毁时候（当前web应用被卸载时候），servlet容器调用此方法。
    public void contextDestroyed(ServletContextEvent servletContextEvent) 

    //ServletContextEvent对象中的一个方法，这个方法能得到ServletContext对象。
    ServletContext getServletContext() 

```


###2.HttpSession
		HttpSessionListener接口
1）监听HttpSession对象被创建或销毁的监听器
2)

```

public class HelloServletContextListener implements HttpSessionListener {

    @Override
    public void sessionCreated(HttpSessionEvent httpSessionEvent) {
    }

    @Override
    public void sessionDestroyed(HttpSessionEvent httpSessionEvent) {
    }
}

```

```
HttpSessionEvent 对象里面的一个方法，可以获得session对象。
    public HttpSession getSession() 

```


###3.ServletRequest
        ServletRequestListener接口
1）监听ServletRequest对象被创建或销毁的监听器
2)

```

public class HelloServletContextListener implements ServletRequestListener {

    @Override
    public void requestDestroyed(ServletRequestEvent servletRequestEvent) {
        
    }

    @Override
    public void requestInitialized(ServletRequestEvent servletRequestEvent) {

    }
}

```


```
 ServletRequestEvent 对象的2个方法：

    public ServletRequest getServletRequest() {
        return this.request;
    }

    public ServletContext getServletContext() {
        return (ServletContext)super.getSource();
    }

```

###4.利用这个3个listener可以研究这3个域对象的生命周期


----------


```
request： 是请求，当一个响应返回时就销毁。

通过超链接从一个jsp页面到另一个jsp页面，request是2个。

通过forward，请求转发从一个jsp页面到另一个jsp页面，request是1个。

同理，在servelt中使用请求转发到一个jsp也是一个请求。

每次刷新页面都会有个一个request创建，request销毁。

重定向是2个request，和转发不一样。



```


----------


```
session：会话，

当第一次访问web应用的一个jsp或者servlet时，且该jsp或sevlet中还需要创建session对象时，此时服务器才会创建session对象。

jsp中如果设置了session=“false”（jsp中不需要session那个隐含对象）的话，这个时候访问这个jsp不会创建session。
如果jsp其中调用了request.getSession(true)这个时候就会创建session。

session销毁，并不是浏览器一关闭session就销毁的。访问的时候能把那个JSESSIONID传给服务器，这个时候不会创建新的session。


session销毁的三种情况：
    session过期。
    调用session的invalidate（）方法。
    当前web应用被卸载（web应用被卸载时候，有个session持久化的问题，会保存到这个session.ser文件中。）

session可以跨页面的。
```


----------

```
application：贯穿于整个web应用的生命周期，当前web应用加载时候创建application对象，卸载时候销毁这个对象。
application 也叫 ServletContext
```

##监听域对象中的属性的增加和删除

监听servletcontext，httpsession，serveltrequest中添加属性，替换属性，移除属性的事件监听器。
每个里面都有3个方法。
这个3个监听器使用的较少，了解。

###  ServletRequestAttributeListener
```
public class AttributeListener implements ServletRequestAttributeListener       {
    

    @Override
    public void attributeAdded(ServletRequestAttributeEvent servletRequestAttributeEvent) {
        System.out.println("向 servletRequest 中添加一个属性");
    }

    @Override
    public void attributeRemoved(ServletRequestAttributeEvent servletRequestAttributeEvent) {
        System.out.println("移除 servletRequest 中一个属性");
    }

    @Override
    public void attributeReplaced(ServletRequestAttributeEvent servletRequestAttributeEvent) {
    }


}

```

API:
```
通过下面三个对象
    ServletRequestAttributeEvent
    HttpSessionBindingEvent
    ServletContextAttributeEvent
中的getName()和getValue()方法就可以获得向对象的属性的名称，属性的值。


```

###HttpSessionAttributeListener
```
public class AttributeListener implements HttpSessionAttributeListener {
    
    @Override
    public void attributeAdded(HttpSessionBindingEvent httpSessionBindingEvent) {
    }

    @Override
    public void attributeRemoved(HttpSessionBindingEvent httpSessionBindingEvent) {
    }

    @Override
    public void attributeReplaced(HttpSessionBindingEvent httpSessionBindingEvent) {
    }
    

}

```

API:
```
通过下面三个对象
    ServletRequestAttributeEvent
    HttpSessionBindingEvent
    ServletContextAttributeEvent
中的getName()和getValue()方法就可以获得向对象的属性的名称，属性的值。


```


###ServletContextAttributeListener
```
public class AttributeListener implements ServletContextAttributeListener {
    
    @Override
    public void attributeAdded(ServletContextAttributeEvent servletContextAttributeEvent) {
        System.out.println("向 servletContext 中添加一个属性");
    }

    @Override
    public void attributeRemoved(ServletContextAttributeEvent servletContextAttributeEvent) {
    }

    @Override
    public void attributeReplaced(ServletContextAttributeEvent servletContextAttributeEvent) {
    }
}

```

API:
```
通过下面三个对象
    ServletRequestAttributeEvent
    HttpSessionBindingEvent
    ServletContextAttributeEvent
中的getName()和getValue()方法就可以获得向对象的属性的名称，属性的值。


```


##监听绑定到httpsession域中的某个对象的状态的事件监听器

###HttpSessionBindingListener
这个不需要在web.xml 中注册配置。

帮助javabean对象了解自己在session域中的状态。

监听实现了该接口的java类对象 绑定到session时候，或者从中解除绑定的事件。

```
package com.马哥私房菜.mvcapp.domain;


import javax.servlet.http.HttpSessionBindingEvent;
import javax.servlet.http.HttpSessionBindingListener;


// 这里个Customer就是一个典型的javabean类。
public class Customer implements HttpSessionBindingListener{
    private Integer id;
    private String name;
    private String address;
    private String phone;

    private String cardType;
    private String cardId;


    public Customer() {
    }

    public Customer(String name, String address, String cardType, String cardId) {
        this.name = name;
        this.address = address;
        this.cardType = cardType;
        this.cardId = cardId;
    }

    public Customer(String name, String address, String phone) {
        this(0, name, address, phone);
    }

    public Customer(Integer id, String name, String address, String phone) {
        this.id = id;
        this.name = name;
        this.address = address;
        this.phone = phone;
    }
    
    @Override
    public void valueBound(HttpSessionBindingEvent httpSessionBindingEvent) {
        //当前对象被绑定到session时调用该方法
        httpSessionBindingEvent.getName();//获取名字，属性名。
        httpSessionBindingEvent.getValue();//获取值，这个值是和this是相等的。
        httpSessionBindingEvent.getSession();//获取session
    }

    @Override
    public void valueUnbound(HttpSessionBindingEvent httpSessionBindingEvent) {
        //解除绑定时调用该方法
    }

    @Override
    public String toString() {
        return "Customer{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", address='" + address + '\'' +
                ", phone='" + phone + '\'' +
                '}';
    }

    public String getCardType() {
        return cardType;
    }

    public void setCardType(String cardType) {
        this.cardType = cardType;
    }

    public String getCardId() {
        return cardId;
    }

    public void setCardId(String cardId) {
        this.cardId = cardId;
    }

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    public String getPhone() {
        return phone;
    }

    public void setPhone(String phone) {
        this.phone = phone;
    }


}

```




```
    HttpSessionBindingEvent对象中的3个方法：
        httpSessionBindingEvent.getName();//获取名字，属性名。
        
        httpSessionBindingEvent.getValue();//获取值，这个值是和this是相等的。
        
        httpSessionBindingEvent.getSession();//获取session
```


###HttpSessionActivationListener

监听  实现了该接口和Serializable接口的java类对象随session钝化和活化 事件

活化，或者钝化，该类要实现Serializable接口，类的序列化
活化指的是从硬盘中读取session对象到内存中。
钝化指的是从内存中写入到硬盘,类的序列化

会存为SESSION.ser 文件

不需要在web.xml 中注册配置。

```

public class Customer implements Serializable, HttpSessionActivationListener {
    @Override
    public void sessionWillPassivate(HttpSessionEvent httpSessionEvent) {
        //从内存中写入到硬盘,该类要实现Serializable接口，类的序列化
    }

    @Override
    public void sessionDidActivate(HttpSessionEvent httpSessionEvent) {
        //从硬盘中读取
    }
}


HttpSessionEvent中的一个方法，httpSessionEvent.getSession();获得session对象
```

##监听器总结

1. 专门用于对其他对象身上发生的事件或状态改变进行监听和相应处理的对象，当被监视的对象发生情况时，立即采取相应的行动。

2. 监听器分类
         1. 监听域对象自身的创建和销毁的事件监听器（ServletContextListener，HttpSessionListener，ServletRequestListener）
         2. 监听域对象中的属性的增加和删除的事件监听器（ServletRequestAttributeListener,   ServletContextAttributeListener, ttpSessionAttributeListener ）
         3. 监听绑定到httpsession域中的某个对象的状态的事件监听器
一共3种类型，8个java类。

3. 如何编写监听器
        编写实现监听器接口的java类
        对应第一种和第二种监听器需要在web.xml文件中进行注册	

4. ServletContextListener最常用，可以在web应用被加载时对web应用的相关资源进行初始化操作（如创建数据库链接池。创建spring的ioc容器。读取当前web应用的初始化参数）
注意这个初始化资源当然也可以在servlet 的init()方法中取初始化，但是这样不专业！

5. 钝化和活化需要实现哪个Serializable接口，（若不实现只能写到硬盘上，不能读取处理！）



##监听器举例
统计在线访客，可以把访客剔除其所在的session。

1. 利用HttpSessionListener可以知道是否有用户访问当前的web应用


```
    @Override
    public void sessionCreated(HttpSessionEvent httpSessionEvent) {
        //这个方法被调用说明有新的访客来
    }

    @Override
    public void sessionDestroyed(HttpSessionEvent httpSessionEvent) {
        //这个方法被调用说明有访客离开
    }
```

2. jsp页面显示当前的访客，显示访客，IP等信息


