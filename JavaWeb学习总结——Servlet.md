**马哥的淘宝店:https://shop592330910.taobao.com/**


>0.总结
>java Servlet 是和平台无关的服务器端组件，它运行在servlet容器中（一般是tomcat，当然也有其他的容器），
>servlet容器负责servelt和客户端的通信以及调用servlet的方法，servlet和客户的通信采用“请求/响应”的模式。

>servlet容器创建和销毁servlet，掌控servlet的生命周期。
>servlet其实就是一个类，一个class而已。
>有一个比较重要的接口，Servlet

#1.创建一个Servlet接口的实现类
实现接口中的所有的方法。
```
public class HelloServlet implements Servlet {
    public HelloServlet() {//构造方法
        System.out.print("hello servlet constructor\n");
    }

    @Override
    public void init(ServletConfig servletConfig) throws ServletException {
        System.out.print("hello servlet init\n");

        //或者servlet配置名称，很少使用, <servlet-name>HelloServlet</servlet-name> 获取的就是这里的自己定义的名
        System.out.print("init servlet name = " + servletConfig.getServletName() + "\n");


        //获取初始化的参数
        Enumeration<String> names = servletConfig.getInitParameterNames();
        while (names.hasMoreElements()) {
            String name = names.nextElement();
            System.out.print("init parameter Name = " + name + "\n");
            System.out.print("init parameter Value= " + servletConfig.getInitParameter(name) + "\n");
        }

        //下面这个是非常关键的一些  servletConfig.getServletContext(), 上下文
        //代表当前web应用的对象

        //1.获取ｗｅｂ应用的初始化参数,
        // 这个和上面的servlet初始化参数的很类似,上面是获取的某个servlet的初始化参数,servlet只有这个servlet可以获取
        // 这里获取的初始化是一个全局的
        ServletContext context = servletConfig.getServletContext();
        names = context.getInitParameterNames();
        while (names.hasMoreElements()) {
            String name = names.nextElement();
            System.out.print("init context parameter Name = " + name + "\n");
            System.out.print("init context parameter Value= " + context.getInitParameter(name) + "\n");
        }


        //2.获取web应用的某一个文件的路径,获取的发布到服务器tomcat 下面的一个绝对路径
        // realPath = /var/lib/tomcat8/webapps/hello/index.jsp
        // 不是部署前的文件的路径
        String realPath = context.getRealPath("/index.jsp");
        System.out.println("init context realPath = " + realPath + "\n");

        //3.获取当前web 应用的名称
        String contextPath = context.getContextPath();
        System.out.println("init context contextPath = " + contextPath + "\n");

        //4.获取当前web 应用的某一个文件输入流
        // context.getResourceAsStream(String path); path 的 / 为当前web 应用的根目录
        //使用classLoader 来获取
        ClassLoader classLoader = getClass().getClassLoader();
        InputStream is = classLoader.getResourceAsStream("");
        System.out.println("1. = " + is);

        //这个是部署后的路径,这里的index.jsp就是相对于根路径下面的
        InputStream is2 = context.getResourceAsStream("index.jsp");
        System.out.println("2. = " + is2);


    }

    @Override
    public ServletConfig getServletConfig() {
        System.out.print("get servlet config\n");
        return null;
    }

    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) 
                throws ServletException, IOException {
        System.out.print("service\n");
    }

    @Override
    public String getServletInfo() {
        System.out.print("get servlet info\n");
        return null;
    }

    @Override
    public void destroy() {
        System.out.print("destroy\n");
    }
}

```

#2.需要在web.xml中配置和映射这个servlet
这里HelloServlet类写完了，不能像之前一样写一个main方法，new出来这个类的实例了，不能这样做了。
servlet的创建是有容器自动的创建了，需要做的就是配置一些，告诉容器哪个类是servlet。

```
    <!-- 添加servlet名称 -->
    <servlet>
        <servlet-name>HelloServlet</servlet-name> <!--servelt 注册名称，名字随便起，但是要起的有意义的名称 -->
        <servlet-class>com.magesfc.javaweb.HelloServlet</servlet-class> <!-- 全类名 -->
       
        <load-on-startup>10</load-on-startup><!-- 这个配置稍后讲解 -->
    </servlet>

    <servlet-mapping> <!-- 映射 -->
        <servlet-name>HelloServlet</servlet-name> <!-- 这个要和某个servlet节点的servlet-name子节点的文本一致 -->
        <url-pattern>/HelloServlet</url-pattern> <!-- 这里的斜杠代表web应用的根目录 -->
    </servlet-mapping>
```


>访问的时候可以使用http://192.168.1.102:8080/HelloServlet（这个是部署到tomcat里面的root下面了，所以没有哪个应用名称）
或者http://192.168.1.102:8080/**应用名称**/HelloServlet

1）一个servlet可以有多个映射。
2）url pattern有2中固定的写法：
可以使用统配符*，*.html 所有html结尾的。带后面扩展名的不能前面再加斜杠了。

```
/*.html 这样写是不合法的。

/* 这样的是合法的。

*.html 这样是合法的。


```




#2.也可以用注解的方式来配置和映射
```
@WebServlet(name = "forwardDestServlet",urlPatterns = {"/forwardDestServlet"})
要特别注意urlpattern这里的的斜杠 / 

```


#3.servlet容器
>1.创建servlet，并调用servlet的相关的生命周期方法（构造方法，init,service,destroy）
>2.运行  jsp, filter, listener,tag 等

#4.servlet生命周期的方法
以下方法都是由容器自动调用的
>构造方法： 第一次请求servlet时候，创建servlet的实例，调用构造器。这说明servlet是单实例的。

>init() 方法： 只被调用一次，在创建好servlet实例后被立即调用。（初始化为什么不在构造器里面做呢？因为init里面有个ServletConfig参数，这是构造器所不具有的）

>service() 方法： 被调用多次，每次请求都会调用service方法，实际用于响应请求的。没刷新一下页面就会调用这个方法。

>destroy() 方法： 被调用一次，当前web应用被卸载的时候被调用，用于释放当前servlet占用的资源，比如数据库链接。


#5.load-on-startup配置参数
可以配置在`<servlet>`节点中
可以指定servlet被创建的时机：若为负数，则在第一次请求时被创建。若为0，正数，则在当前servlet被容器加载时创建实例。且数值越小越早被创建。

```
    <servlet>
        <servlet-name>HelloServlet</servlet-name>                             <!--servelt 注册名称，名字随便起，但是要起的有意义的名称 -->
        <servlet-class>com.magesfc.javaweb.HelloServlet</servlet-class>       <!-- 全类名 -->
        <load-on-startup>1</load-on-startup>                                  <!-- 这个配置稍后讲解 -->
    </servlet>


```

#6.ServletConfig  
这个是一个接口，是有tomcat来实现的。
servlet配置，封装相关配置信息，并且可以获取servlet context对象（这个对象比较重要）

```
public interface ServletConfig {
    String getServletName();

    ServletContext getServletContext();

    String getInitParameter(String var1);

    Enumeration<String> getInitParameterNames();
}
```
##1）获取初始化参数
首先要配置servlet的初始化参数

```
    <servlet>
        <servlet-name>validateColorServlet</servlet-name>
        <servlet-class>com.magesfc.mvcapp.servlet.ValidateColorServlet</servlet-class>
        <init-param>
            <param-name>user</param-name>    <!--这个是初始化参数的名称 -->
            <param-value>mtest</param-value> <!--这个是值-->
        </init-param>
        <init-param>
            <param-name>passwd</param-name>
            <param-value>123456</param-value>
        </init-param>
    </servlet>

```

这个节点```<init-param>``` 配置到servlet节点中，这个节点必须在load-on-startup节点前面

然后再来获取初始化参数

```
    String getInitParameter(String name);//获取指定名称的初始化参数值
    Stirng user = servletConfig.getInitParameter("user");

    Enumeration<String> getInitParameterNames();//获取多个初始化参数名称，返回一个enum类型。能获取名字就能获取到对应的值。
    //获取初始化的参数
    Enumeration<String> names = servletConfig.getInitParameterNames();
    while (names.hasMoreElements()) {
        String name = names.nextElement();
        String value = servletConfig.getInitParameter(name);
        System.out.print("init parameter Name = " + name + "\n");
        System.out.print("init parameter Value= " + value + "\n");
    }
```

##2)获取servlet的名字
 String getServletName();
用的很少

```
//或者servlet配置名称，很少使用, <servlet-name>HelloServlet</servlet-name> 获取的就是这里的自己定义的名
System.out.print("init servlet name = " + servletConfig.getServletName() + "\n");

```

##3）获取servlet context上下文对象
 ServletContext getServletContext();
这个很重要
servlet引擎为每一个web应用程序都创建一个对应的servletcontext对象，该对象被包含在servletconfig对象中，调用getServletContext（）方法就可以得到。
由于一个web应用中的所有servlet都共享同一个servletconfig对象，所以，servletcontext对象被称之为application对象（web应用对象，全局只有一个实例的。）
可以任务是当前web应用的大管家，可以从前获取到web应用的各个方面的信息。


#7.重点介绍ServletContext

##1）.获取当前web应用的初始化参数

>context里面的初始化参数和servlet里面的初始化参数的区别，context里面的是全局的，每个servlet都可以使用。

首先在web.xml中要配置：

```
<!--获取web应用的初始化参数 -->
<context-param>
    <param-name>driver</param-name>
    <param-value>com.mysql.jdbc.Driver</param-value>
</context-param>

<context-param>
    <param-name>jdbcUrl</param-name>
    <param-value>jdbc:mysql://com.mysql.jdbc</param-value>
</context-param>
```

然后是在代码里面获取：
```
//下面这个是非常关键的一些  servletConfig.getServletContext(), 上下文
//代表当前web应用的对象

//1.获取web应用的初始化参数,
// 这个和上面的servlet初始化参数的很类似,上面是获取的某个servlet的初始化参数,servlet只有这个servlet可以获取
// 这里获取的初始化是一个全局的
ServletContext context = servletConfig.getServletContext();
String jdbcUrl = context.getInitParameter("jdbcUrl");//单独获取一个指定名称的初始化参数。

names = context.getInitParameterNames();//这个和之前的servletconfig的获取初始化参数的用法很类似。这个是获取所有的名称，获取了名称就可以获取到对应的值了。
while (names.hasMoreElements()) {
    String name = names.nextElement();
    System.out.print("init context parameter Name = " + name + "\n");
    System.out.print("init context parameter Value= " + context.getInitParameter(name) + "\n");
}


```



##2）.获取web应用的某一个文件的路径

获取的发布到服务器tomcat 下面的一个绝对路径getRealPath（）方法。

```
// 是这个  realPath = /var/lib/tomcat8/webapps/hello/index.jsp
// 而不是部署前的文件的路径
String realPath = context.getRealPath("/index.jsp");
System.out.println("init context realPath = " + realPath + "\n");
```

##3）.获取当前web 应用的名称
```
String contextPath = context.getContextPath();
System.out.println("init context contextPath = " + contextPath + "\n");

//一般是访问的时候，url中端口号8080后面的一个名称


```

##4）.获取当前web 应用的某一个文件输入流
```

// context.getResourceAsStream(String path); path 的 / 为当前web 应用的根目录

//使用classLoader 来获取
ClassLoader classLoader = getClass().getClassLoader();
InputStream is = classLoader.getResourceAsStream("index.jsp");
System.out.println("1. = " + is);

//这个是部署后的路径,这里的index.jsp就是相对于根路径下面的
InputStream is2 = context.getResourceAsStream("index.jsp");
System.out.println("2. = " + is2);


```





ServletContext接口的源代码
```
public interface ServletContext {
    String TEMPDIR = "javax.servlet.context.tempdir";
    String ORDERED_LIBS = "javax.servlet.context.orderedLibs";

    String getContextPath();

    ServletContext getContext(String var1);

    int getMajorVersion();

    int getMinorVersion();

    int getEffectiveMajorVersion();

    int getEffectiveMinorVersion();

    String getMimeType(String var1);

    Set<String> getResourcePaths(String var1);

    URL getResource(String var1) throws MalformedURLException;

    InputStream getResourceAsStream(String var1);

    RequestDispatcher getRequestDispatcher(String var1);

    RequestDispatcher getNamedDispatcher(String var1);

    /** @deprecated */
    Servlet getServlet(String var1) throws ServletException;

    /** @deprecated */
    Enumeration<Servlet> getServlets();

    /** @deprecated */
    Enumeration<String> getServletNames();

    void log(String var1);

    /** @deprecated */
    void log(Exception var1, String var2);

    void log(String var1, Throwable var2);

    String getRealPath(String var1);

    String getServerInfo();

    String getInitParameter(String var1);

    Enumeration<String> getInitParameterNames();

    boolean setInitParameter(String var1, String var2);

    Object getAttribute(String var1);

    Enumeration<String> getAttributeNames();

    void setAttribute(String var1, Object var2);

    void removeAttribute(String var1);

    String getServletContextName();

    Dynamic addServlet(String var1, String var2);

    Dynamic addServlet(String var1, Servlet var2);

    Dynamic addServlet(String var1, Class<? extends Servlet> var2);

    <T extends Servlet> T createServlet(Class<T> var1) throws ServletException;

    ServletRegistration getServletRegistration(String var1);

    Map<String, ? extends ServletRegistration> getServletRegistrations();

    javax.servlet.FilterRegistration.Dynamic addFilter(String var1, String var2);

    javax.servlet.FilterRegistration.Dynamic addFilter(String var1, Filter var2);

    javax.servlet.FilterRegistration.Dynamic addFilter(String var1, Class<? extends Filter> var2);

    <T extends Filter> T createFilter(Class<T> var1) throws ServletException;

    FilterRegistration getFilterRegistration(String var1);

    Map<String, ? extends FilterRegistration> getFilterRegistrations();

    SessionCookieConfig getSessionCookieConfig();

    void setSessionTrackingModes(Set<SessionTrackingMode> var1);

    Set<SessionTrackingMode> getDefaultSessionTrackingModes();

    Set<SessionTrackingMode> getEffectiveSessionTrackingModes();

    void addListener(String var1);

    <T extends EventListener> void addListener(T var1);

    void addListener(Class<? extends EventListener> var1);

    <T extends EventListener> T createListener(Class<T> var1) throws ServletException;

    JspConfigDescriptor getJspConfigDescriptor();

    ClassLoader getClassLoader();

    void declareRoles(String... var1);
}
```

#9. GET 请求和 POST 请求:

##1). 使用GET方式传递参数:

>①. 在浏览器地址栏中输入某个URL地址或单击网页上的一个超链接时，浏览器发出的HTTP请求消息的请求方式为GET。 
>②. 如果网页中的<form>表单元素的 method 属性被设置为了“GET”，浏览器提交这个FORM表单时生成的HTTP请求消息的请求方式也为GET。 
>③. 使用GET请求方式给WEB服务器传递参数的格式：  

```
http://www.magesfc.com/counter.jsp?name=lc&password=123
```

④. 使用GET方式传送的数据量一般限制在 1KB 以下。 

##2). 使用 POST 方式传递参数:

>①. POST 请求方式主要用于向 WEB 服务器端程序提交 FORM 表单中的数据: form 表单的 method 置为 POST
>②. POST 方式将各个表单字段元素及其数据作为 HTTP 消息的实体内容发送给 WEB 服务器，传送的数据量要比使用GET方式传送的数据量大得多。  

```
POST /counter.jsp HTTP/1.1
referer: http://localhost:8080/Register.html
content-type: application/x-www-form-urlencoded
host: localhost:8080
content-length: 43

name=zhangsan&password=123              --请求体中传递参数. 
```

#10.如何在servlet中获取请求信息
##1）servlet的service（）方法
用于应答请求，因为每次请求都会调用service（）方法

```
    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {
        System.out.print("service\n");
    }
```
ServletRequest 这个类封装了请求的信息，可以从中获取到任何请求的信息。
ServletResponse 这个类封装了响应的信息，
这个方法是容器自动调用的，参数也是容器自动传过来的。不需要我们总结调用，传递参数。

```
org.apache.catalina.connector.RequestFacade@75cd61e6
org.apache.catalina.connector.ResponseFacade@71f2fb87
```
##2）ServletRequest
获取请求参数的4个方法

```

<form action="loginServlet3" method="get">
    user: <input type="text" name="username"/>
    password: <input type="password" name="password"/>

    <input type="submit" value="submit"/>

    <br/>

    <input type="checkbox", name="interesting" value="reading"/>reading
    <input type="checkbox", name="interesting" value="game"/>game
    <input type="checkbox", name="interesting" value="shop"/>shop
    <input type="checkbox", name="interesting" value="run"/>run


</form>
```

```
        //String getParameter(String var1);//获取请求参数，根据参数名，得到值
        String username = servletRequest.getParameter("username");
        String password = servletRequest.getParameter("password");


        //String[] getParameterValues(String var1);//获取请求参数，根据参数名，得到值的一个数组，在多选的时候会是一组值。
        //若请求参数有多个值，如果还有改方法getParameter（）只能获取到第一个提交的值！！！
        String[] interesting = servletRequest.getParameterValues("interesting");
        
        //Map<String, String[]> getParameterMap();//获取请求参数，得到一个map对象
        Map<String, String[]> map = servletRequest.getParameterMap();
        for (Map.Entry<String, String[]> entry : map.entrySet()) {
            System.out.println("servletRequest: map = " + entry);
        }


        //Enumeration<String> getParameterNames();//获取请求参数名称
        Enumeration<String> names = servletRequest.getParameterNames();
        while (names.hasMoreElements()) {
            String name = names.nextElement();
            System.out.print("servletRequest parameter Name = " + name + "\n");
            System.out.print("servletRequest parameter Value= " + servletRequest.getParameter(name) + "\n");
        }


```

##3)还可以获取其他相关信息
，HttpServletRequest是ServletRequest的一个子接口，专门针对http请求定义的。
这里需要进行一个强制转换
```
        String uri = ((HttpServletRequest) servletRequest).getRequestURI();
        System.out.println("uri = " + uri);
        
        StringBuffer url = ((HttpServletRequest) servletRequest).getRequestURL();
        System.out.println("url = " + url);
        
        String method = ((HttpServletRequest) servletRequest).getMethod();
        System.out.println("method = " + method);
        
        String queryString = ((HttpServletRequest) servletRequest).getQueryString();
        System.out.println("query string= " + queryString);//post请求的话这个值是null
        
        String servletPath = ((HttpServletRequest) servletRequest).getServletPath();
        System.out.println("servletPath = " + servletPath);//映射路径

        //这个是和属性相关的方法，获取特定名称的属性，这个很重要，后面讲解！！！        
        Object attr = ((HttpServletRequest) servletRequest).getAttribute("attr");
```
上述代码的输出，这样访问，
`http://localhost:8080/hello/helloServlet?username=test&password=123&interesting=game&interesting=shop&interesting=run`

```
uri = /hello/helloServlet
url = http://localhost:8080/hello/helloServlet
method = GET
query string= username=test&password=123&interesting=game&interesting=shop&interesting=run
servletPath = /helloServlet
```

##4）ServletResponse

servletResponse.getWriter()方法，这个常用一些。
```
http://localhost:8080/hello/helloServlet


    PrintWriter out = servletResponse.getWriter();
    out.println("hello world");//可以把字符串打印到客户端的浏览器上
```

servletResponse.setContentType（）方法，设置返回的内容类型。
```
        servletResponse.setContentType("application/msword");//这个设置了响应内容类型为一个微软的word类型
```

servletResponse.getOutputStream()方法，这个和文件下载会用到
```
        ServletOutputStream outputstream = servletResponse.getOutputStream();
```

HttpServletResponse是一个ServletResponse的子接口，这个子接口里面也有很多方法。
这个是和重定向相关的方法，很重要，后面详细介绍
```
        ((HttpServletResponse) servletResponse).sendRedirect("path");
```


#11.GenericServlet类
自己实现一个GenericServlet类，实现Servlet, ServletConfig 接口
这样写自己的servlet时候直接继承这个抽象方法，这样会简单一些，而不像之前一样每个接口里面的方法都有实现。
实现ServletConfig接口的好处是，可以在servelt里面直接调用ServletConfig里面的方法。
```

public abstract class GenericServlet implements Servlet, ServletConfig {
    private ServletConfig mServletConfig;
    private ServletContext mServletContext;


    @Override
    public void init(ServletConfig servletConfig) throws ServletException {
        mServletConfig = servletConfig;
        mServletContext = servletConfig.getServletContext();
        this.init();

    }

    protected abstract void init();//这里为什么会多一个init方法呢？？？？？

    @Override
    public ServletConfig getServletConfig() {
        return mServletConfig;
    }

    @Override
    public void service(ServletRequest servletRequest, ServletResponse servletResponse) throws ServletException, IOException {

    }

    @Override
    public String getServletInfo() {
        return null;
    }

    @Override
    public void destroy() {

    }

    @Override
    public String getServletName() {
        return mServletConfig.getServletName();//本身我们是没有能力来实现这个方法，我们借助于ServletConfig来实现这个方法，这里有装饰器设计模式的概念。
    }

    @Override
    public ServletContext getServletContext() {
        return mServletContext;//本身我们是没有能力来实现这个方法，我们借助于ServletConfig来实现
    }

    @Override
    public String getInitParameter(String s) {
        return mServletConfig.getInitParameter(s);//本身我们是没有能力来实现这个方法，我们借助于ServletConfig来实现
    }

    @Override
    public Enumeration<String> getInitParameterNames() {
        return mServletConfig.getInitParameterNames() ;//本身我们是没有能力来实现这个方法，我们借助于ServletConfig来实现
    }
}

```

protected abstract void init();//这里为什么会多一个init方法呢？？？？？
```
//对当前的servlet进行初始化，需要覆盖父类的init(ServletConfig servletConfig) 方法。这样就会把
        mServletConfig = servletConfig;
        mServletContext = servletConfig.getServletContext();
//这个2个变量的初始化给覆盖掉了，这个初始化就会不起作用了，容易报空指针异常的。
//这里提供一个空的init（）方法，就是让继承的类覆盖这个方法，进行自己的初始化。
```

```
类关系
接口Servelt， ServletConfig, Serializable
    抽象类GenericServlet
        HttpServlet
```

#12.HttpServlet类
针对于http协议定义的一个servlet基类，后续写servlet都可以基于这个类创建直接的servlet。
继承这个HttpServlet类，覆盖doPost（）方法或doGet（）方法
```
public abstract class HttpServlet extends GenericServlet implements Serializable {

    public HttpServlet() {
    }

    protected void doGet(HttpServletRequest req, HttpServletResponse resp) 
                                    throws ServletException, IOException {
    }

    protected void doPost(HttpServletRequest req, HttpServletResponse resp)
                                     throws ServletException, IOException {
    }
 

    protected void service(HttpServletRequest req, HttpServletResponse resp) 
                                    throws ServletException, IOException {
        String method = req.getMethod();
        if (method.equals("GET")) {
            this.doGet(req, resp);
        } else if (method.equals("POST")) {
            this.doPost(req, resp);
        }
    }

    public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
        HttpServletRequest request;
        HttpServletResponse response;
        try {
            request = (HttpServletRequest)req;
            response = (HttpServletResponse)res;
        } catch (ClassCastException var6) {
            throw new ServletException("non-HTTP request or response");
        }

        this.service(request, response);
    }
}

```
