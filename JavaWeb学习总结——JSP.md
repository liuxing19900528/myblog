**马哥的淘宝店:https://shop592330910.taobao.com/**


JSP：java server page,  java服务器端页面，在html中编写java代码。


#1.为什么会有jsp
jsp是同servlet编写的一种技术，它将java代码和html代码语句混合在同一个文件中编写
只对网页中的要动态产生的内容采用java代码编写，而对固定不变的静态内容采用普通的html代码编写。

#2.jsp的hello程序
新建jsp页面，在 `<%   %>`   之间编写java代码即可。

这个jsp文件可以放在web应用的任何目录（除了WEB-INFO目录）

#3.jsp的运行原理

```
$ ls  /tomcat/_day_29/work/Catalina/localhost/hello/org/apache/jsp/ -lh                                            

total 32K
-rw-rw-r-- 1 mamh mamh 6.1K 8月   1 16:30 hello2_jsp.class
-rw-rw-r-- 1 mamh mamh 5.3K 8月   1 16:30 hello2_jsp.java
-rw-rw-r-- 1 mamh mamh 7.0K 8月  29 13:41 index_jsp.class
-rw-rw-r-- 1 mamh mamh 6.9K 8月  29 13:41 index_jsp.java

```
jsp的本质上其实是一个servlet。每个jsp页面第一次被访问的时候，jsp引擎将他翻译为一个servlet。然后接着把这个servlet源代码程序编译为class文件。
```
public final class index_jsp extends org.apache.jasper.runtime.HttpJspBase
    implements org.apache.jasper.runtime.JspSourceDependent,
                 org.apache.jasper.runtime.JspSourceImports {

  private static final javax.servlet.jsp.JspFactory _jspxFactory =
          javax.servlet.jsp.JspFactory.getDefaultFactory();

  private static java.util.Map<java.lang.String,java.lang.Long> _jspx_dependants;

  private static final java.util.Set<java.lang.String> _jspx_imports_packages;

  private static final java.util.Set<java.lang.String> _jspx_imports_classes;

  static {
    _jspx_imports_packages = new java.util.HashSet<>();
    _jspx_imports_packages.add("javax.servlet");
    _jspx_imports_packages.add("javax.servlet.http");
    _jspx_imports_packages.add("javax.servlet.jsp");
    _jspx_imports_classes = new java.util.HashSet<>();
    _jspx_imports_classes.add("com.atguigu.javaweb.HelloServlet");
    _jspx_imports_classes.add("com.atguigu.javaweb.Person");
  }

  private volatile javax.el.ExpressionFactory _el_expressionfactory;
  private volatile org.apache.tomcat.InstanceManager _jsp_instancemanager;

  public java.util.Map<java.lang.String,java.lang.Long> getDependants() {
    return _jspx_dependants;
  }

  public java.util.Set<java.lang.String> getPackageImports() {
    return _jspx_imports_packages;
  }

  public java.util.Set<java.lang.String> getClassImports() {
    return _jspx_imports_classes;
  }

  public javax.el.ExpressionFactory _jsp_getExpressionFactory() {
    if (_el_expressionfactory == null) {
      synchronized (this) {
        if (_el_expressionfactory == null) {
          _el_expressionfactory = _jspxFactory.getJspApplicationContext(getServletConfig().getServletContext()).getExpressionFactory();
        }
      }
    }
    return _el_expressionfactory;
  }

  public org.apache.tomcat.InstanceManager _jsp_getInstanceManager() {
    if (_jsp_instancemanager == null) {
      synchronized (this) {
        if (_jsp_instancemanager == null) {
          _jsp_instancemanager = org.apache.jasper.runtime.InstanceManagerFactory.getInstanceManager(getServletConfig());
        }
      }
    }
    return _jsp_instancemanager;
  }

  public void _jspInit() {
  }

  public void _jspDestroy() {
  }

  public void _jspService(final javax.servlet.http.HttpServletRequest request, final javax.servlet.http.HttpServletResponse response)
        throws java.io.IOException, javax.servlet.ServletException {

final java.lang.String _jspx_method = request.getMethod();
if (!"GET".equals(_jspx_method) && !"POST".equals(_jspx_method) && !"HEAD".equals(_jspx_method) && !javax.servlet.DispatcherType.ERROR.equals(request.getDispatcherType())) {
response.sendError(HttpServletResponse.SC_METHOD_NOT_ALLOWED, "JSPs only permit GET POST or HEAD");
return;
}

    final javax.servlet.jsp.PageContext pageContext;
    javax.servlet.http.HttpSession session = null;
    final javax.servlet.ServletContext application;
    final javax.servlet.ServletConfig config;
    javax.servlet.jsp.JspWriter out = null;
    final java.lang.Object page = this;
    javax.servlet.jsp.JspWriter _jspx_out = null;
    javax.servlet.jsp.PageContext _jspx_page_context = null;


    try {
      response.setContentType("text/html;charset=UTF-8");
      pageContext = _jspxFactory.getPageContext(this, request, response,
                  null, true, 8192, true);
      _jspx_page_context = pageContext;
      application = pageContext.getServletContext();
      config = pageContext.getServletConfig();
      session = pageContext.getSession();
      out = pageContext.getOut();
      _jspx_out = out;

      out.write("\n");
      out.write("\n");
      out.write("\n");
      out.write("<html>\n");
      out.write("<head>\n");
      out.write("    <title>$Title$</title>\n");
      out.write("</head>\n");
      out.write("<body>\n");
      out.write("<a href=\"upload.jsp\">upload</a>\n");
      out.write("<br/><br/>\n");
      out.write("<a href=\"download.jsp\">download</a>\n");
      out.write("\n");



    //9 个隐含对象
    //1.HttpServletRequest类的一个对象
    String username = request.getParameter("username");
    System.out.println(username);

    //2.HttpServletResponse类的一个对象,这个几乎不会调用
    Class clazz = response.getClass();
    System.out.println(clazz);

    //3.PageContext,页面的上下文，重要的一个对象。
    // 可以从改对象中获取页面所有信息，可以获取其他的８个隐含对象(后面学习自定义标签使用它)
    HttpServletRequest req = (HttpServletRequest) pageContext.getRequest();
    System.out.println(req == request);

    //4.session,HttpSession类的一个对象，代表浏览器和服务器的一次会话
    System.out.println("session ID = " + session.getId());


    //5.application,代表当前web应用，可以获取web应用的初始化参数
    System.out.println(application.getInitParameter("user"));


    //6.config,当前jsp对象的servlet的ServletConfig对象，几乎不用
    System.out.println(config.getInitParameter("hellojsp.password"));


    //7.out, JspWriter　对象　可以直接调用out.println(),可以把字符串都打印到页面上
    out.println(config.getInitParameter("hellojsp.password"));
    out.print("<br/>");
    out.println("session ID = " + session.getId());


    //8.page是Obejct的对象，等于是this,    final java.lang.Object page = this;
    //指向当前ｊｓｐ对象的一个引用,但是他是一个Object类型只能使用Object　类的方法，几乎不用
    out.print("<br/>");
    out.print(this);
    out.print("<br/>");
    out.print(page);

    //9.exception  只有在error 页面才可以使用



      out.write("\n");
      out.write("\n");
      out.write("\n");
      out.write("</body>\n");
      out.write("</html>\n");
      out.write("\n");
      out.write("\n");
      out.write("</body>\n");
      out.write("</html>\n");
    } catch (java.lang.Throwable t) {
      if (!(t instanceof javax.servlet.jsp.SkipPageException)){
        out = _jspx_out;
        if (out != null && out.getBufferSize() != 0)
          try {
            if (response.isCommitted()) {
              out.flush();
            } else {
              out.clearBuffer();
            }
          } catch (java.io.IOException e) {}
        if (_jspx_page_context != null) _jspx_page_context.handlePageException(t);
        else throw new ServletException(t);
      }
    } finally {
      _jspxFactory.releasePageContext(_jspx_page_context);
    }
  }
}

```

#4.jsp中的9个隐含对象

>pageContext, request, session, application(对属性作用域的范围从小到大)这几个很重要
>out,  response,  config,   page这几个不常用

```
public void _jspService(final javax.servlet.http.HttpServletRequest request, final javax.servlet.http.HttpServletResponse response)
        throws java.io.IOException, javax.servlet.ServletException {

    final javax.servlet.jsp.PageContext pageContext;
    javax.servlet.http.HttpSession session = null;
    final javax.servlet.ServletContext application;
    final javax.servlet.ServletConfig config;
    javax.servlet.jsp.JspWriter out = null;
    final java.lang.Object page = this;
    javax.servlet.jsp.JspWriter _jspx_out = null;
    javax.servlet.jsp.PageContext _jspx_page_context = null;


    try {
      pageContext = _jspxFactory.getPageContext(this, request, response,null, true, 8192, true);
      application = pageContext.getServletContext();
      config = pageContext.getServletConfig();
      session = pageContext.getSession();
      out = pageContext.getOut();

      //这下面是我们自己的代码
```

##1）.request
HttpServletRequest类的一个对象

```
    String username = request.getParameter("username");
    System.out.println(username);
```

##2）.response
HttpServletResponse类的一个对象,这个几乎不会调用

```
    Class clazz = response.getClass();
    System.out.println(clazz);
```

##3）.pageContext
,页面的上下文，重要的一个对象。

```
    // 可以从该对象中获取页面所有信息，可以获取其他的８个隐含对象(后面学习自定义标签使用它)
    HttpServletRequest req = (HttpServletRequest) pageContext.getRequest();
    System.out.println(req == request);
```

##4）.session
HttpSession类的一个对象，代表浏览器和服务器的一次会话，

```
    System.out.println("session ID = " + session.getId());
```

##5）.application
代表当前web应用，可以获取web应用的初始化参数
它就是    ServletContext 的对象。
```
    System.out.println(application.getInitParameter("user"));
```

##6）.config
当前jsp对象的servlet的ServletConfig对象，几乎不用
在web.xml 中配置一下这个初始化参数
```
    <servlet>
        <servlet-name>hellojsp</servlet-name>
        <jsp-file>/index.jsp</jsp-file>
        <init-param>
            <param-name>hellojsp.password</param-name>
            <param-value>hellojsp1234569273498271394723094723984134</param-value>
        </init-param>
    </servlet>
    
    <servlet-mapping>
        <servlet-name>hellojsp</servlet-name>
        <url-pattern>/index.jsp</url-pattern>
    </servlet-mapping>
```
在index.jsp文件中获取初始化参数

```
    System.out.println(config.getInitParameter("hellojsp.password"));
```
访问需要访问映射的那个地址

```
http://localhost:8080/hello/indexjsp
```

##7）.out
 JspWriter　对象　可以直接调用out.println(),可以把字符串都打印到页面上

```
    out.println(config.getInitParameter("hellojsp.password"));
    out.print("<br/>");
    out.println("session ID = " + session.getId());
```

##8）.page
是Obejct的对象，等于是this,    final java.lang.Object page = this;

```
    //指向当前jsp对象的一个引用,但是他是一个Object类型只能使用Object　类的方法，几乎不用
    out.print("<br/>");
    out.print(this);
    out.print("<br/>");
    out.print(page);
```

```
this:    org.apache.jsp.index_jsp@4368ebb3
page:    org.apache.jsp.index_jsp@4368ebb3 
```

##9).exception
  只有在error 页面才可以使用

只有主动声明这个页面是isErrorPage=true的页面上才可以使用

```
<%@ page isErrorPage="true" %>
```

#5.jsp语法

##1）jsp模板元素
>jsp页面中的静态html元素称之位模板元素

##2）jsp表达式（重要内容）

```
<%
    //1.jsp 表达式可以将java 变量或java 表达式直接在页面上面输出
    Date date = new Date();
    out.print(date);      //可以这样写  
%>
```

```
<%= date %><!-- 还可以这样写 前提是要有这个date变量，这个就是jsp的表达式 -->

```

##3） jsp脚本片段

```
位于jsp页面中的<%   %>之间的代码就是脚本片段
```

```

<%
    String ageStr = request.getParameter("age");
    Integer age = Integer.parseInt(ageStr);

    if (age >= 18) {
        out.print("成人");
    } else {
        out.print("未成年");
    }
%>
```
也可以像下面这样来写，这就是所谓的脚本片段。多个脚本片段可以互相访问。
脚本片段的代码最后会翻译到jspService（）方法里面。
```
<%
    String ageStr = request.getParameter("age");
    int age = 0;
    try {
        age = Integer.parseInt(ageStr);
    } catch (Exception e) {

    }
    if (age >= 18) {
%>
        成人
<%
    } else {
%>
        未成年
<%
    }


%>

上面的代码最后会被翻译为：
    String ageStr = request.getParameter("age");
    int age = 0;
    try {
        age = Integer.parseInt(ageStr);
    } catch (Exception e) {

    }
    if (age >= 18) {
      out.write("\n");
      out.write("成人\n");
    } else {
      out.write("\n");
      out.write("未成年\n");
    }
```
##4）jsp声明
一般不用。
```
jsp声明将java代码封装在<%!   %>之间，它里面的代码将被插入到servlet的jspService（）方法的外面。
```

```
如果在hello2.jsp 里面加入下面代码
<%!
    void test(){
        System.out.println("this is test method");
    }

%>

上面的声明会在类里面生成一个方法的。会在hello2_jsp这个类里面有这个方法。

public final class hello2_jsp extends org.apache.jasper.runtime.HttpJspBase

    void test() {
        System.out.println("this is test method");
    }
}
```
##5）注释
> jsp注释可以阻止java代码的执行。
```
<!--  这个是html注释-->
<%-- 这个jsp注释   --%>  
```


#6.和属性相关的方法

1） 四个方法
```
    public abstract void setAttribute(String var1, Object var2);

    public abstract Object getAttribute(String var1);

    public abstract void removeAttribute(String var1);

    Enumeration<String> getAttributeNames();
```
2）pageContext,request,session,application 这4个对象都有这几个属性的方法
>pageContext 属性的作用范围仅限于当前jsp页面。
>request 属性的作用范围仅限于同一个请求。（在请求转发的情况下可以跨页面获取属性的）
>session 属性的作用范围仅限于一次会话，
>application 属性的作用范围限于当前web应用，是范围最大的，只要在一处设置，其他各个地方都可以获取到。
```
<%
    //只有这４个可以设置属性，这４个称之为域对象，设置属性
    //属性作用范围仅仅限于当前页面
    pageContext.setAttribute("p", "pageContextValue");

    //仅限于同一个请求
    request.setAttribute("r", "requestValue");

    //作用范围限于一次会话,浏览器打开直道关闭，在此期间会话不失效的前提下。
    session.setAttribute("s", "sessionValue");
    
    //限于当前web应用，全局的范围，范围最大
    application.setAttribute("a", "applicationValue");
%>

<br><br>
pageContext: <%= pageContext.getAttribute("p") %>
<br><br>
requestContext: <%= request.getAttribute("r") %>
<br><br>
sessionContext: <%= session.getAttribute("s") %>
<br><br>
applicationContext: <%= application.getAttribute("a") %>
<br><br>


```

#7.请求转发和重定向

```

@WebServlet(name = "forwardServlet", urlPatterns = {"/forwardServlet"})
public class ForwardServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("forward servlet do get ....");

        //转发这里的属性可以传递给下个servlet，应为请求转发是一个request
        request.setAttribute("r", "forward servlet");
        System.out.println("forward servlet do get ...." + request.getAttribute("r"));


        //请求的转发
        //1.调用HttpServeltRequest 的 getRequestDispatcher(addr) 方法，传入要转发的地址
        String path = "forwardDestServlet";
        RequestDispatcher requestDispatcher = request.getRequestDispatcher(path);

        //2.调用HttpServeltRequest 的 forward() 进行请求的转发
        requestDispatcher.forward(request, response); //这样就转发到forwardDestServlet这里了

    }
}
```

```
@WebServlet(name = "forwardDestServlet",urlPatterns = {"/forwardDestServlet"})
public class ForwardDestServlet extends HttpServlet {
    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

    }

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        System.out.println("forward dest servlet do get ....");

        System.out.println("ForwardDestServlet servlet do get ...." + request.getAttribute("r"));

        
        PrintWriter out = response.getWriter();
        out.println("dest page.....");

    }
}
```

请求转发和重定向的区别
```
请求转发和重定向的区别      
    请求转发地址栏不发生变化.
    转发是一个请求,
    请求转发只能发到当前web应用的某个资源,不能转发出去.
    对转发而言斜杠代表的是当前应用的根目录.

    重定向会改变的.
    重定向是２个请求.
    重定向可以到任何资源.
    对于重定向斜杠代表是站点的/

在这个基础上来讲mvc设计模式（请求转发）

```


#8.jsp指令
指令放在`<%@  %>`  直接，有3个执行，page，taglib，include

多个属性可以写到一个指令里面，也可以分开写到多个指令中。
page指令的作用都在整个jsp页面，一般放到jsp页面的开头几行。

```
<%@ page 属性名=“属性值” %>

<%@ page contentType="text/html;charset=UTF-8" language="java" %>

<%@taglib prefix=""%>

<%@include file=""%>
```

1）page指令

```
<%@page import="java.lang.Integer" %> 指定导入的java类。
```

```
<%@page session="true|false" %> 作用是配置是否可以使用session隐含对象。
访问当前页面是否要生成httpsession对象。
```

```
<%@ page errorPage="/error.jsp" %>
<%@ page isErrorPage="true" %>
从当前页面出错跳转到error.jsp的时候 内部其实用的是转发。

如何是客户端不能直接访问某个页面，对于tomcat来说，web-info下面的文件是不能直接访问的，隐私文件。可以通过转发来访问web-info下面的隐私文件。
```

```
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
contentType指定当前页面的内容类型，也可以指定编码
实际会被翻译为：      response.setContentType("text/html;charset=UTF-8");
通常情况下取值为text/html;charset=UTF-8
```

```
<%@page pageEncoding="utf-8" %>指定当前jsp页面的编码，通常和contentType里面一致。
```

```
<%@page isELIgnored="true" %> 指定当前jsp页面是否可以使用EL表达式，通常取值true。
```

2）include指令
include会被文件的所有内容包含进来的。静态包含，源码级别包含。
翻译后是一个java源文件。
相对路径问题，斜杠开头表示当前web应用的根目录。

```
String str= "";
<%@include file="attr1.jsp"%>
include之前声明的变量在包含的那个attr1.jsp文件中是可以使用的。这个在动态包含中是不生效的。
```
3）jsp：include 标签
```
动态包含，这个会生成2个java文件的。
<jsp:include page="attr1.jsp"/>

翻译为这样的一个语句：org.apache.jasper.runtime.JspRuntimeLibrary.include(request, response, "attr1.jsp", out, false);


      
```

#9.jsp标签

```
<jsp:include page="attr1.jsp"/>
使用这种标签的方式可以使用子标签<jsp:param> 像下一个jsp页面传入一些参数。
```

```
在jsp页面转发，直接这样写标签，不用再写java代码了。
<jsp:forward page="attr1.jsp" />
翻译为：
      if (true) {
        _jspx_page_context.forward("attr1.jsp");
        return;
      }

使用这种标签的方式可以使用子标签<jsp:param> 像下一个jsp页面传入一些参数。
<jsp:forward page="attr1.jsp">
    <jsp:param name="username" value="xxxx"/>
</jsp:forward>
在attr1.jsp里面可以使用request.getParameter("username")获取到请求的参数。
```



