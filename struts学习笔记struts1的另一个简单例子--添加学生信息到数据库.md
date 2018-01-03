马哥的淘宝店:https://shop592330910.taobao.com/



总结：
>1.action什么时候初始化，也就是什么时候创建Action的对象？发出该action请求时创建实例对象。不是读配置时创建的。
>2.每个action只会创建一个实例对象，也就是单例的。
>3.action是线程不安全的，因为所有的请求共享一个action实例。

>4.怎样实现action的安全编程：
>>注意不要用实例变量或者类变量来 共享   只 是针对某个请求的数据。
>>注意资源操作的同步性。


----------
ActionMapping，ActionForward，ActionForm

>action-mappings节点下面定义各个action节点。
>action-mappings元素帮助进行框架内部流程控制，可将请求URI映射到Action类，
>将Action对象与ActionForm相关联。
>这个下面可以设置多个子节点action节点。

>action节点，所描述的是特定的请求路径和一个相应的action类之间的映射关系。

```
<action-mappings>
    <action path="/login" type="com.mamh.struts1.demo.action.LoginAction" name="loginForm" >
        <forward name="loginSuccess" path="/loginSuccess.jsp"/>
        <forward name="loginFailure" path="/loginFailure.jsp"/>
    </action>

    <action path="/addStudentAction" type="com.mamh.struts1.demo.action.AddStudentAction" name="addStudentForm">
        <forward name="addSuccess" path="/addSuccess.jsp"/>
        <forward name="addFailure" path="/addFailure.jsp"/>
    </action>
</action-mappings>
```
>  
>   
>execute()方法第一个参数就是ActionMapping对象，我们可以通过mapping获取到一些值。
>其实获取的就是action-mappings节点下面的所有的配置内容。
>getName() Return name of the form bean, if any, associated with this Action.
```

System.out.println("=====================================================");

String name = mapping.getName();//addStudentForm
String path = mapping.getPath();
String type = mapping.getType();
String[] forwards = mapping.findForwards();
for(String fname: forwards){
    System.out.println(fname);
}
System.out.println(name);// addStudentForm
System.out.println(path);// //addStudentAction
System.out.println(type);// com.mamh.struts1.demo.action.AddStudentAction
System.out.println("=====================================================");
```

>```  <forward name="loginFailure" path="/loginFailure.jsp" redirect="false"/>```
>这里的redirect默认是false，是表示转发。设置为true表示重定向。这个值可以有true，false，yes，no这四个。

>全局跳转global-forwards，减少代码量，实现代码共享。
```
    <global-forwards>
        <forward name="error" path="/error.jsp"/>
    </global-forwards>
```


action节点下面的属性
>`<action path="/login" type="com.mamh.struts1.demo.action.LoginAction" name="loginForm" scope="request" >`
>>validate="true" 属性, 判断是否需要校验，用于是否校验。不校验就不会去调用validate()方法。这个值可以有true，false，yes，no这4个。默认是true。
> >input="loginFailure.jsp"，该Action所引用的Form校验失败跳转到这个input对应的页面。跳转其实用的是转发的方式。一般结合validate="true" 属性一起使用。
>>scope="request" 属性, action节点可以设置一个作用范围：scope这个可以取值request，session中的1个。默认是session。
>>attribute="addStudentForm" 属性，设置和Action关联的ActionForm在request，session内的属性key，通过request，session的getAttribute()方法返回该ActionForm实例。`request.getSession().getAttribute("addStudentForm");代码中可以这样去取`能不能通过这种方式获取到ActionForm关键是这个决定的。这个如果不设置默认去name的值。
>>name="addStudentForm"属性,

>查找action，看action中是否有name属性，scope属性，根据name和scope找到对应的ActionForm。

>找到就用现成的ActionForm对象，如果没找到就新建ActionForm对象（实例化，并且存到相应的scope里面）。

>调用ActionForm的reset（）方法，复位。

>取值（从客户参数，request.getParamter()取得参数，）和赋值（设置ActionForm属性，调用setter方法）。

>如果action节点的validate设置了true就做校验，调用ActionForm的validate（）方法。
>>校验成功，派发请求到相应的action。
>>校验失败，失败一般跳到错误页面。

>执行相应的action的execute()方法。

```
先调用无参数的构造方法：AddStudentForm() construtor必须要有一个无参数的构造器！！！
保存到相应的scope(request,session)中。我们可以通过一个 AttributeListener 来研究这个执行过程。
然后调用reset（）方法：public void reset(ActionMapping mapping, HttpServletRequest request)
最后调用setter（）之类的方法，setter方法调用顺序不重要的。
    setBirth()
    setName()
    setScore()
    setMajor()
调用Action的execute（）方法。
```

```
<form-beans>
    <form-bean name="loginForm" type="com.mamh.struts1.demo.form.LoginForm" />
    <form-bean name="addStudentForm" type="com.mamh.struts1.demo.form.AddStudentForm"/>
</form-beans>
```
>在execute()方法中分析ActionForm
>自己从session中取这个ActionForm对象和execute方法中那个参数form是一个对象的。是==的。
```
    @Override
    public ActionForward execute(ActionMapping mapping, ActionForm form,
                                 HttpServletRequest request, HttpServletResponse response) throws Exception {
        System.out.println("AddStudentAction execute(");
        AddStudentForm sessionForm = (AddStudentForm) request.getSession().getAttribute("addStudentForm");
        AddStudentForm addStudentForm = (AddStudentForm) form;
        System.out.println("sessionForm == addStudentForm   ： "+ (sessionForm == addStudentForm));
    ｝
```
>对于调用的setter方法，这个方法名是和form表单中的相一至的。不关心属性名称是否一致。
>> 比如form中有个`<input type="text" name="name"/>`就会调用setName()方法。
>>比如form中有个`<input type="text" name="major"/>`就会调用setMajor()方法。


















----------
**下面看一个例子，添加学生信息到数据库**

首先新建个AddStudentAction 的Action的子类，重写了execute()方法，
这个方法里面调用操作数据库的相关的业务逻辑代码，然后根据业务逻辑返回的
值来决定最后要转发到哪个页面。
```
public class AddStudentAction extends Action {
    @Override
    public ActionForward execute(ActionMapping mapping, ActionForm form,
                                 HttpServletRequest request, HttpServletResponse response) throws Exception {
        System.out.println("AddStudentAction execute(");
        AddStudentForm addStudentForm = (AddStudentForm) form;
        DAO studentDao = new StudentDAOImpl();
        boolean addResult = studentDao.add(addStudentForm);
        String url = "";
        if (addResult) {
            url = "addSuccess";
        } else {
            url = "addFailure";
        }
        return mapping.findForward(url);
    }
}

```
这里定义一个dao的接口
```
public interface DAO {
    boolean add(AddStudentForm form);
}

```

实现添加数据到student数据库表的StudentDAOImpl 类
```
public class StudentDAOImpl implements DAO {
    public boolean add(AddStudentForm form) {
        String sql = "insert into student(name,birth,major,score) values(?, ?, ?,?)";
        JdbcUtils.update(sql, form.getName(), form.getBirth(), form.getMajor(), form.getScore());
        return true;
    }
}

```
数据库相关的工具类方法
```
public class JdbcUtils {
    /**
     * 执行 SQL 语句, 使用 PreparedStatement
     *
     * @param sql
     * @param args: 填写 SQL 占位符的可变参数
     */
    public static void update(String sql, Object... args) {
        Connection connection = null;
        PreparedStatement preparedStatement = null;

        try {
            connection = JdbcUtils.getConnection();
            preparedStatement = connection.prepareStatement(sql);

            for (int i = 0; i < args.length; i++) {
                preparedStatement.setObject(i + 1, args[i]);
            }

            preparedStatement.executeUpdate();

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JdbcUtils.release(null, preparedStatement, connection);
        }
    }


    public static void release(Statement statement, PreparedStatement preparedStatement, Connection connection) {
        //4.关闭statement
        try {
            if (statement != null)
                statement.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }

        //5.关闭数据库链接
        try {
            if (connection != null)
                connection.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }


    /**
     * 编写一个通用的方法，在不修改源程序的情况下，可以获取任何数据库的链接
     * 解决方法：把数据库驱动Driver的全类名，url，user，password放入到一个
     * 配置文件中。
     *
     * @return
     */
    public static Connection getConnection() throws Exception {
        //1. Class.getResourceAsStream(String path) ：
        // path 不以’/'开头时默认是从此类所在的包下取资源，
        // 以’/'开头则是从ClassPath根下获取。其只是通过path构造一个绝对路径，最终还是由ClassLoader获取资源。
        com.mysql.jdbc.Driver driver = null;
        InputStream in = JdbcUtils.class.getResourceAsStream("/jdbc.prop");

        Properties properties = new Properties();
        properties.load(in);

        String className = "com.mysql.jdbc.Driver";
        String jdbcUrl = "jdbc:mysql://10.0.63.43/test";
        String user = "test";
        String password = "123456";

        className = properties.getProperty("className");
        jdbcUrl = properties.getProperty("jdbcUrl");
        user = properties.getProperty("user");
        password = properties.getProperty("password");

        System.out.println("className = " + className);
        System.out.println("jdbcUrl   = " + jdbcUrl);
        System.out.println("user      = " + user);
        System.out.println("password  = " + password);

        Class.forName(className);


        Connection c = DriverManager.getConnection(jdbcUrl, user, password);

        return c;
    }
}

```

新建一个AddStudentForm 的类，继承ActionForm 。里面的属性名对应jsp页面表单中的input的name的值。
```

public class AddStudentForm extends ActionForm {
    private String name = null;
    private String major = null;
    private String score = null;
    private String birth = null;

    public AddStudentForm() {
        System.out.println("AddStudentForm(1) construtor!!!!!!!!!!");
    }

    @Override
    public ActionErrors validate(ActionMapping mapping, ServletRequest request) {
        System.out.println("    public ActionErrors validate(ActionMapping mapping, ServletRequest request) {");
        return super.validate(mapping, request);
    }

    @Override
    public ActionErrors validate(ActionMapping mapping, HttpServletRequest request) {
        System.out.println("    public ActionErrors validate(ActionMapping mapping, HttpServletRequest request) {");
        return super.validate(mapping, request);
    }

    @Override
    public void reset(ActionMapping mapping, ServletRequest request) {
        System.out.println("public void reset(ActionMapping mapping, ServletRequest request) ");
        super.reset(mapping, request);
    }

    @Override
    public void reset(ActionMapping mapping, HttpServletRequest request) {
        System.out.println("public void reset(ActionMapping mapping, HttpServletRequest request)");
        super.reset(mapping, request);
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        System.out.println("setName()");
        this.name = name;
    }

    public String getMajor() {
        return major;
    }

    public void setMajor(String major) {
        System.out.println("setMajor()");
        this.major = major;
    }

    public String getScore() {
        return score;
    }

    public void setScore(String score) {
        System.out.println("setScore()");
        this.score = score;
    }

    public String getBirth() {
        return birth;
    }

    public void setBirth(String birth) {
        System.out.println("setBirth()");
        this.birth = birth;
    }
}

```
struts-config.xml 文件添加form-bean节点。同时添加action 节点。这个对应了上面的AddStudentForm，和AddStudentAction类
```
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE struts-config PUBLIC
        "-//Apache Software Foundation//DTD Struts Configuration 1.2//EN"
        "http://struts.apache.org/dtds/struts-config_1_2.dtd">

<struts-config>
    <form-beans>
        <form-bean name="loginForm" type="com.mamh.struts1.demo.form.LoginForm" />
        <form-bean name="addStudentForm" type="com.mamh.struts1.demo.form.AddStudentForm" />
    </form-beans>
    <action-mappings>
        <action path="/login" type="com.mamh.struts1.demo.action.LoginAction" name="loginForm" >
            <forward name="loginSuccess" path="/loginSuccess.jsp"/>
            <forward name="loginFailure" path="/loginFailure.jsp"/>
        </action>

        <action path="/addStudentAction" type="com.mamh.struts1.demo.action.AddStudentAction" name="addStudentForm">
            <forward name="addSuccess" path="/addSuccess.jsp"/>
            <forward name="addFailure" path="/addFailure.jsp"/>
        </action>
    </action-mappings>

</struts-config>
```
web.xml 文件
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
添加学生信息的jsp，里面含有表单form。
```
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>addStudentAction</title>
</head>
<body>

<form action="/addStudentAction.do" method="post">
    name: <label>
    <input type="text" name="name"/>
</label>
    <br/>

    major: <label>
    <input type="text" name="major"/>
</label>
    <br/>

    score: <label>
    <input type="text" name="score"/>
</label>
    <br/>

    birth: <label>
    <input type="text" name="birth"/>
</label>
    <br/>

    <input type="submit" value="add student"/>

</form>
</body>
</html>

```

```
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>addFailure.jsp</title>
</head>
<body>
<H1>add failure</H1>
</body>
</html>
```

```
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>addSuccess.jsp</title>
</head>
<body>
<H1>add success!!!!!!!!!!!!</H1>
</body>
</html>
```


----------
用于研究ActionForm什么时候被设置到相应的scope中的。
```
public class AttributeListener implements
        ServletRequestAttributeListener, HttpSessionAttributeListener {
    public void attributeAdded(ServletRequestAttributeEvent event) {
        String name = event.getName();
        Object value = event.getValue();
        System.out.println("attributeAdded(ServletRequestAttributeEvent event) { name = " + name + " , value = " + value);
    }

    public void attributeRemoved(ServletRequestAttributeEvent event) {
        String name = event.getName();
        Object value = event.getValue();
        System.out.println("attributeRemoved(ServletRequestAttributeEvent event) { name = " + name + " , value = " + value);

    }

    public void attributeReplaced(ServletRequestAttributeEvent event) {
        String name = event.getName();
        Object value = event.getValue();
        System.out.println("attributeReplaced(ServletRequestAttributeEvent event) { name = " + name + " , value = " + value);

    }

    public void attributeAdded(HttpSessionBindingEvent event) {
        String name = event.getName();
        Object value = event.getValue();
        System.out.println("attributeAdded(HttpSessionBindingEvent event) { name = " + name + " , value = " + value);
    }

    public void attributeRemoved(HttpSessionBindingEvent event) {
        String name = event.getName();
        Object value = event.getValue();
        System.out.println("attributeRemoved(HttpSessionBindingEvent event) { name = " + name + " , value = " + value);
    }

    public void attributeReplaced(HttpSessionBindingEvent event) {
        String name = event.getName();
        Object value = event.getValue();
        System.out.println("attributeReplaced(HttpSessionBindingEvent event) { name = " + name + " , value = " + value);
    }
}

```

----------
下面介绍一些idea的技巧

```
Intellij IDEA中有很多快捷键让人爱不释手，stackoverflow上也有一些有趣的讨论。
每个人都有自己的最爱，想排出个理想的榜单还真是困难。以前也整理过Intellij的快捷键，
这次就按照我日常开发时的使用频率，简单分类列一下我最喜欢的十大快捷-神-键吧。

1 智能提示

Intellij首当其冲的当然就是Intelligence智能！基本的代码提示用Ctrl+Space，还有更
智能地按类型信息提示Ctrl+Shift+Space，但因为Intellij总是随着我们敲击而自动提示，
所以很多时候都不会手动敲这两个快捷键(除非提示框消失了)。用F2/ Shift+F2移动到有错误
的代码，Alt+Enter快速修复(即Eclipse中的Quick Fix功能)。当智能提示为我们自动补全
方法名时，我们通常要自己补上行尾的反括号和分号，当括号嵌套很多层时会很麻烦，这时我们
只需敲Ctrl+Shift+Enter就能自动补全末尾的字符。而且不只是括号，例如敲完if/for时也
可以自动补上{}花括号。
最后要说一点，Intellij能够智能感知spring、hibernate等主流框架的配置文件和类，以
静制动，在看似“静态”的外表下，智能地扫描理解你的项目是如何构造和配置的。


2 重构

Intellij重构是另一完爆Eclipse的功能，其智能程度令人瞠目结舌，比如提取变量时自动检查
到所有匹配同时提取成一个变量等。尤其看过《重构-改善既有代码设计》之后，有了Intellij的
配合简直是令人大呼过瘾！也正是强大的智能和重构功能，使Intellij下的TDD开发非常顺畅。
切入正题，先说一个无敌的重构功能大汇总快捷键Ctrl+Shift+Alt+T，叫做Refactor This。
按法有点复杂，但也符合Intellij的风格，很多快捷键都要双手完成，而不像Eclipse不少最有
用的快捷键可以潇洒地单手完成(不知道算不算Eclipse的一大优点)，但各位用过Emacs的话就会
觉得也没什么了(非Emacs黑)。此外，还有些最常用的重构技巧，因为太常用了，若每次都在
Refactor This菜单里选的话效率有些低。比如Shift+F6直接就是改名，Ctrl+Alt+V则是提取变量。

3 代码生成

这一点类似Eclipse，虽不是独到之处，但因为日常使用频率极高，所以还是罗列在榜单前面。
常用的有fori/sout/psvm+Tab即可生成循环、System.out、main方法等boilerplate样板代
码，用Ctrl+J可以查看所有模板。后面“辅助”一节中将会讲到Alt+Insert，在编辑窗口中点击可
以生成构造函数、toString、getter/setter、重写父类方法等。这两个技巧实在太常用了，几
乎每天都要生成一堆main、System.out和getter/setter。
另外，Intellij IDEA 13中加入了后缀自动补全功能(Postfix Completion)，比模板生成更
加灵活和强大。例如要输入for(User user : users)只需输入user.for+Tab。再比如，要输
入Date birthday = user.getBirthday();只需输入user.getBirthday().var+Tab即可。


4 编辑

编辑中不得不说的一大神键就是能够自动按语法选中代码的Ctrl+W以及反向的Ctrl+Shift+W了。
此外，Ctrl+Left/Right移动光标到前/后单词，Ctrl+[/]移动到前/后代码块，这些类Vim风格
的光标移动也是一大亮点。以上Ctrl+Left/Right/[]加上Shift的话就能选中跳跃范围内的代码。
Alt+Forward/Backward移动到前/后方法。还有些非常普通的像Ctrl+Y删除行、Ctrl+D复制行、
Ctrl+</>折叠代码就不多说了。
关于光标移动再多扩展一点，除了Intellij本身已提供的功能外，我们还可以安装ideaVim或者
emacsIDEAs享受到Vim的快速移动和Emacs的AceJump功能(超爽！)。另外，Intellij的书签功
能也是不错的，用Ctrl+Shift+Num定义1-10书签(再次按这组快捷键则是删除书签)，然后通过
Ctrl+Num跳转。这避免了多次使用前/下一编辑位置Ctrl+Left/Right来回跳转的麻烦，而且此
快捷键默认与Windows热键冲突(默认多了Alt，与Windows改变显示器显示方向冲突，一不小心显
示器就变成倒着显式的了，冏啊)。


5 查找打开

类似Eclipse，Intellij的Ctrl+N/Ctrl+Shift+N可以打开类或资源，但Intellij更加智能一
些，我们输入的任何字符都将看作模糊匹配，省却了Eclipse中还有输入*的麻烦。最新版本的IDEA
还加入了Search Everywhere功能，只需按Shift+Shift即可在一个弹出框中搜索任何东西，包
括类、资源、配置项、方法等等。
类的继承关系则可用Ctrl+H打开类层次窗口，在继承层次上跳转则用Ctrl+B/Ctrl+Alt+B分别对
应父类或父方法定义和子类或子方法实现，查看当前类的所有方法用Ctrl+F12。
要找类或方法的使用也很简单，Alt+F7。要查找文本的出现位置就用Ctrl+F/Ctrl+Shift+F在当
前窗口或全工程中查找，再配合F3/Shift+F3前后移动到下一匹配处。
Intellij更加智能的又一佐证是在任意菜单或显示窗口，都可以直接输入你要找的单词，
Intellij就会自动为你过滤。



6 其他辅助

以上这些神键配上一些辅助快捷键，即可让你的双手90%以上的时间摆脱鼠标，专注于键盘仿佛在进行钢琴表演。这些不起眼却是至关重要的最后一块拼图有：
Ø  命令：Ctrl+Shift+A可以查找所有Intellij的命令，并且每个命令后面还有其快捷键。所以它不仅是一大神键，也是查找学习快捷键的工具。
Ø  新建：Alt+Insert可以新建类、方法等任何东西。
Ø  格式化代码：格式化import列表Ctrl+Alt+O，格式化代码Ctrl+Alt+L。
Ø  切换窗口：Alt+Num，常用的有1-项目结构，3-搜索结果，4/5-运行调试。Ctrl+Tab切换标签页，Ctrl+E/Ctrl+Shift+E打开最近打开过的或编辑过的文件。
Ø  单元测试：Ctrl+Alt+T创建单元测试用例。
Ø  运行：Alt+Shift+F10运行程序，Shift+F9启动调试，Ctrl+F2停止。
Ø  调试：F7/F8/F9分别对应Step into，Step over，Continue。
此外还有些我自定义的，例如水平分屏Ctrl+|等，和一些神奇的小功能Ctrl+Shift+V粘贴很早以前拷贝过的，Alt+Shift+Insert进入到列模式进行按列选中。


7 最终榜单

这榜单阵容太豪华了，后几名都是如此有用，毫不示弱。
Ø  Top #10切来切去：Ctrl+Tab
Ø  Top #9选你所想：Ctrl+W
Ø  Top #8代码生成：Template/Postfix +Tab
Ø  Top #7发号施令：Ctrl+Shift+A
Ø  Top #6无处藏身：Shift+Shift
Ø  Top #5自动完成：Ctrl+Shift+Enter
Ø  Top #4创造万物：Alt+Insert

太难割舍，前三名并列吧！
Ø  Top #1智能补全：Ctrl+Shift+Space
Ø  Top #1自我修复：Alt+Enter
Ø  Top #1重构一切：Ctrl+Shift+Alt+T
```
