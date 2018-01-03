**马哥的淘宝店:https://shop592330910.taobao.com/**

#1.读取配置（初始化ModuelConfig对象）
struts框架总控制器（ActionServlet）是一个Servlet，在web.xml中配置成自动启动的servlet。
读取配置文件（struts-config.xml）的配置信息，为不同的struts模块初始化相应的ModuleConfig对象
ActionConfig，ControlConfig，FormBeanConfig，ForwardConfig，

```
    <servlet>
        <servlet-name>action</servlet-name>
        <servlet-class>org.apache.struts.action.ActionServlet</servlet-class>
        <init-param>
            <param-name>config</param-name>
            <param-value>/WEB-INF/struts-config.xml</param-value>
        </init-param>
        <load-on-startup>0</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>action</servlet-name>
        <url-pattern>*.do</url-pattern>
    </servlet-mapping>
```
马哥的淘宝店:https://shop592330910.taobao.com/
#2.发送请求
用户体检表单或调用url向web应用容器提交一个请求，请求用的http协议

#3.填充FORM（实例化，复位，填充数据，校验，保存）
（*.do请求）从ActionConfig中找出对应请求的action子类。
如果没有对应的action，控制器直接转发给jsp或者静态页面。
如果有对应的action且这个action有一个相应的ActionForm，ActionForm被实例化并用http请求的数据
填充其属性，并保存在servlet context中（request或serssion中）这样他们就可以被其他Action对象或
jsp调用。
ActionServelt去填充数据，数据来自http请求中，
马哥的淘宝店:https://shop592330910.taobao.com/
#4.派发请求
控制器根据配置信息ActionConfig讲请求派发到具体的Action，相应的FormBean一并传给这个Action的
execute（）方法。
为什么能找到正确的Action呢？是根据配置文件的。

```
    <action-mappings>
        <action path="/login" type="com.mamh.struts1.demo.action.LoginAction" name="loginForm" >
            <forward name="loginSuccess" path="/loginSuccess.jsp"/>
            <forward name="loginFailure" path="/loginFailure.jsp"/>
        </action>
    </action-mappings>
```
马哥的淘宝店:https://shop592330910.taobao.com/
#5.处理业务
Action一般只包含一个execute（）方法，他负责执行相应的页面逻辑（调用其他业务模块）。
完毕返回一个ActionForward对象，控制器通过该ActionForward对象来进行转发工作。

```
    @Override
    public ActionForward execute(ActionMapping mapping, ActionForm form, HttpServletRequest request, HttpServletResponse response) throws Exception {
        System.out.println("LoginAction execute(");
        LoginForm loginForm = (LoginForm)form;

        if(loginForm.getUsername().equals("mamh")){
            return mapping.findForward("loginSuccess");
        }   else {
            return mapping.findForward("loginFailure");
        }
    }
```
马哥的淘宝店:https://shop592330910.taobao.com/
#6.返回响应
Action根据业务处理的不同结果返回一个目标响应对象给总控制器，该目标响应对象对应一个具体的jsp或
另一个Action。

```
            return mapping.findForward("loginFailure");
```
马哥的淘宝店:https://shop592330910.taobao.com/
#7.查找响应（翻译响应）
总控制器根据业务功能Action返回的目标响应对象，找到对应的资源对象，通常是一个具体的jsp页面或
另一个Action。

#8.响应用户
目标响应对象将结果展现给用户目标响应对象（jsp）将结果页面展现给用户。jsp是在服务器上执行完把结果返回给用户。

马哥的淘宝店:https://shop592330910.taobao.com/



使用idea新建一个maven的web工程

pom.xml 文件
```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.mamh</groupId>
    <artifactId>struts1demo</artifactId>
    <packaging>war</packaging>
    <version>1.0-SNAPSHOT</version>
    <name>struts1demo Maven Webapp</name>
    <url>http://maven.apache.org</url>
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>3.8.1</version>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>struts</groupId>
            <artifactId>struts</artifactId>
            <version>1.2.9</version>
        </dependency>

        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>2.8.2</version>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
            <version>2.5</version>
        </dependency>


    </dependencies>
    <build>
        <finalName>struts1demo</finalName>
    </build>
</project>

```
马哥的淘宝店:https://shop592330910.taobao.com/
web.xml文件
```
<!DOCTYPE web-app PUBLIC
        "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
        "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
    <display-name>Archetype Created Web Application</display-name>
    <servlet>
        <servlet-name>action</servlet-name>
        <servlet-class>org.apache.struts.action.ActionServlet</servlet-class>
        <init-param>
            <param-name>config</param-name>
            <param-value>/WEB-INF/struts-config.xml</param-value>
        </init-param>
        <load-on-startup>0</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>action</servlet-name>
        <url-pattern>*.do</url-pattern>
    </servlet-mapping>
</web-app>

```
马哥的淘宝店:https://shop592330910.taobao.com/
struts-config.xml 文件
```
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE struts-config PUBLIC
        "-//Apache Software Foundation//DTD Struts Configuration 1.2//EN"
        "http://struts.apache.org/dtds/struts-config_1_2.dtd">

<struts-config>
    <form-beans>
        <form-bean name="loginForm" type="com.mamh.struts1.demo.form.LoginForm" />


    </form-beans>
    <action-mappings>
        <action path="/login" type="com.mamh.struts1.demo.action.LoginAction" name="loginForm" >
            <forward name="loginSuccess" path="/loginSuccess.jsp"/>
            <forward name="loginFailure" path="/loginFailure.jsp"/>
        </action>
    </action-mappings>

</struts-config>
```


马哥的淘宝店:https://shop592330910.taobao.com/
LoginAction 文件

这里特别注意重新的那个方法的参数，注意使用的是http参数的那个execute（）方法，特别注意
```

public class LoginAction extends Action {

    public LoginAction() {
        super();
    }

    @Override
    public ActionForward execute(ActionMapping mapping, ActionForm form, HttpServletRequest request, HttpServletResponse response) throws Exception {
        System.out.println("LoginAction execute(");
        LoginForm loginForm = (LoginForm)form;

        if(loginForm.getUsername().equals("mamh")){
            return mapping.findForward("loginSuccess");
        }   else {
            return mapping.findForward("loginFailure");
        }
    }
}

```
马哥的淘宝店:https://shop592330910.taobao.com/
LoginForm 文件
```
public class LoginForm extends ActionForm {
    private String username;
    private String password;


    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}

```

login.jsp 文件
```
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>login.jsp</title>
</head>
<body>

<form action="/login.do" method="post">
    username: <label>
    <input type="text" name="username"/>
</label>
    <br/>
    password: <label>
    <input type="password" name="password"/>
</label>
    <br/>

    <input type="submit" value="login"/>

</form>
</body>
</html>

```

```
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>loginFailure.jsp</title>
</head>
<body>
<H1>登录失败</H1>
</body>
</html>
```

```
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>loginSuccess.jsp</title>
</head>
<body>
<H1>登录成功</H1>

</body>
</html>
```
马哥的淘宝店:https://shop592330910.taobao.com/
