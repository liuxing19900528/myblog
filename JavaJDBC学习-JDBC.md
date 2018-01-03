马哥的淘宝店:https://shop592330910.taobao.com/



```
JDBC(Java Database Connectivity)是一个独立于特定数据库管理系统、
通用的SQL数据库存取和操作的公共接口（一组API），
定义了用来访问数据库的标准Java类库，使用这个类库可以以一种标准的方法、方便地访问数据库资源
JDBC为访问不同的数据库提供了一种统一的途径，为开发者屏蔽了一些细节问题。
JDBC的目标是使Java程序员使用JDBC可以连接任何提供了JDBC驱动程序的数据库系统，
这样就使得程序员无需对特定的数据库系统的特点有过多的了解，从而大大简化和加快了开发过程。

```

```
try {
    Driver driver = new com.mysql.jdbc.Driver();

    String url = "jdbc:mysql://localhost/test";
    Properties info = new Properties();
    info.put("user", "test");
    info.put("password", "123456");

    Connection c = driver.connect(url, info);

    System.out.println(c);

} catch (SQLException e) {
    e.printStackTrace();
}

```


 编写一个通用的方法，在不修改源程序的情况下，可以获取任何数据库的链接
解决方法：把数据库驱动Driver的全类名，url，user，password放入到一个
配置文件中。
     
```


    public Connection getConnection() throws Exception {
        String className = null;
        String jdbcUrl = null;
        String user = null;
        String password = null;

        InputStream in = getClass().getResourceAsStream("jdbc.prop");
        Properties properties = new Properties();
        properties.load(in);

        className = properties.getProperty("className");
        jdbcUrl = properties.getProperty("jdbcUrl");
        user = properties.getProperty("user");
        password = properties.getProperty("password");


        Driver driver = (Driver) Class.forName(className).newInstance();

        Properties info = new Properties();
        info.put("user", user);
        info.put("password", password);

        Connection c = driver.connect(jdbcUrl, info);
        return c;
    }
```


DriverManager,driver的管理类，一般开发中用这个，不直接使用driver。
Java.sql.Driver 接口是所有 JDBC 驱动程序需要实现的接口。这个接口是提供给数据库厂商使用的，不同数据库厂商提供不同的实现
在程序中不需要直接去访问实现了 Driver 接口的类，而是由驱动程序管理器类(java.sql.DriverManager)去调用这些Driver实现

```
对于 Oracle 数据库连接，采用如下形式： 
    jdbc:oracle:thin:@localhost:1521:sid
对于 SQLServer 数据库连接，采用如下形式：
    jdbc:microsoft:sqlserver//localhost:1433; DatabaseName=sid
对于 MYSQL 数据库连接，采用如下形式：   
    jdbc:mysql://localhost:3306/sid

```
        String className = null;驱动的全类名
        String jdbcUrl = null;
        String user = null;
        String password = null;
```
   public void testDriverManager(){
        //driver的管理类，一般开发中用这个，不直接使用driver。
        //driver的管理类，一般开发中用这个，不直接使用driver。
        String className = null;
        String jdbcUrl = null;
        String user = null;
        String password = null;

        InputStream in = getClass().getResourceAsStream("jdbc.prop");
        Properties properties = new Properties();
        properties.load(in);

        className = properties.getProperty("className");
        jdbcUrl = properties.getProperty("jdbcUrl");
        user = properties.getProperty("user");
        password = properties.getProperty("password");


        /*
         * 
         * 加载 JDBC 驱动需调用 Class 类的静态方法 forName()，向其传递要加载的 JDBC 驱动的类名
         *DriverManager 类是驱动程序管理器类，负责管理驱动程序
         *通常不用显式调用 DriverManager 类的 registerDriver() 方法来注册驱动程序类的实例，
         *因为 Driver 接口的驱动程序类都包含了静态代码块，在这个静态代码块中，
         *会调用 DriverManager.registerDriver() 方法来注册自身的一个实例
         *
         */
        Class.forName(className);

        //可以调用 DriverManager 类的 getConnection() 方法建立到数据库的连接
        //JDBC URL 用于标识一个被注册的驱动程序，驱动程序管理器通过这个 URL 选择正确的驱动程序，从而建立到数据库的连接。
        //JDBC URL的标准由三部分组成，各部分间用冒号分隔。 
        //jdbc:<子协议>:<子名称>
        //协议：JDBC URL中的协议总是jdbc 
        //子协议：子协议用于标识一个数据库驱动程序
        //子名称：一种标识数据库的方法。子名称可以依不同的子协议而变化，用子名称的目的是为了定位数据库提供足够的信息 
        Connection connection = DriverManager.getConnection(jdbcUrl, user, password);
        
    }
```


statement的使用

```
通过调用 Connection 对象的 createStatement 方法创建该对象
该对象用于执行静态的 SQL 语句，并且返回执行结果
Statement 接口中定义了下列方法用于执行 SQL 语句：
ResultSet excuteQuery(String sql)
int excuteUpdate(String sql)

```

```

    @Test
    public void testStatement() {
        //1.获取数据库链接
        Connection connection = null;
        Statement statement = null;
        try {
            connection = getConnection();

            //2.准备插入的sql语句
            String insertsql = "insert into customers(name, address, phone) values('abc','aaa','bbbb')";


            //3.执行插入。获取操作sql的statement对象
            // 调用statement的 executeUpdate(sql)方法,这个可以执行insert，update，delete，但不能是select。
            statement = connection.createStatement();

            statement.executeUpdate(insertsql);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {


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

    }

```

ResultSet的使用

```
通过调用 Statement 对象的 excuteQuery() 方法创建该对象
ResultSet 对象以逻辑表格的形式封装了执行数据库操作的结果集，ResultSet 接口由数据库厂商实现
ResultSet 对象维护了一个指向当前数据行的游标，初始的时候，游标在第一行之前，可以通过 ResultSet 对象的 next() 方法移动到下一行
ResultSet 接口的常用方法：
boolean next()
getString()

```

```

    /**
     * ResultSet: 结果集. 封装了使用 JDBC 进行查询的结果. 
     * 1. 调用 Statement 对象的 executeQuery(sql) 可以得到结果集.
     * 2. ResultSet 返回的实际上就是一张数据表. 有一个指针指向数据表的第一样的前面.
     * 可以调用 next() 方法检测下一行是否有效. 若有效该方法返回 true, 且指针下移. 相当于
     * Iterator 对象的 hasNext() 和 next() 方法的结合体
     * 3. 当指针对位到一行时, 可以通过调用 getXxx(index) 或 getXxx(columnName)
     * 获取每一列的值. 例如: getInt(1), getString("name")
     * 4. ResultSet 当然也需要进行关闭. 
     */
    @Test
    public void testResultSet(){
        //获取 id=4 的 customers 数据表的记录, 并打印
        
        Connection conn = null;
        Statement statement = null;
        ResultSet rs = null;
        
        try {
            //1. 获取 Connection
            conn = JDBCTools.getConnection();
            System.out.println(conn);
            
            //2. 获取 Statement
            statement = conn.createStatement();
            System.out.println(statement);
            
            //3. 准备 SQL
            String sql = "SELECT id, name, email, birth " +
                    "FROM customers";
            
            //4. 执行查询, 得到 ResultSet
            rs = statement.executeQuery(sql);
            System.out.println(rs);
            
            //5. 处理 ResultSet
            while(rs.next()){
                int id = rs.getInt(1);
                String name = rs.getString("name");
                String email = rs.getString(3);
                Date birth = rs.getDate(4);
                
                System.out.println(id);
                System.out.println(name);
                System.out.println(email);
                System.out.println(birth);
            }
            
        } catch (Exception e) {
            e.printStackTrace();
        } finally{
            //6. 关闭数据库资源. 
            JDBCTools.release(rs, statement, conn);
        }
        
    }

```


通用的一个 更新方法
```
    /**
     * 通用的一个 更新方法，包括insert，update，delete
     *
     * @param sql
     */
    public void update(String sql) {
        //1.获取数据库链接
        Connection connection = null;
        Statement statement = null;
        try {
            connection = getConnection();
            statement = connection.createStatement();
            statement.executeUpdate(sql);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
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
    }
```

通过上面的代码我们发现很多重复的代码，像获取链接，关闭资源等，这些我们可以放到一个utils类里面，工具类方法。

```
//工具类方法
public class JdbcTools {

    public static void release(Statement statement, Connection connection) {
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
        InputStream in = getClass().getResourceAsStream("jdbc.prop");
        Properties properties = new Properties();
        properties.load(in);

        className = properties.getProperty("className");
        jdbcUrl = properties.getProperty("jdbcUrl");
        user = properties.getProperty("user");
        password = properties.getProperty("password");


        Driver driver = (Driver) Class.forName(className).newInstance();

        Properties info = new Properties();
        info.put("user", user);
        info.put("password", password);

        Connection c = driver.connect(jdbcUrl, info);
        return c;
    }
}

```

SQL 注入攻击

```
SQL 注入是利用某些系统没有对用户输入的数据进行充分的检查，而在用户输入数据中注入非法的 SQL 语句段或命令，从而利用系统的 SQL 引擎完成恶意行为的做法
对于 Java 而言，要防范 SQL 注入，只要用 PreparedStatement 取代 Statement 就可以了

```

PreparedStatement

```
可以通过调用 Connection 对象的 preparedStatement() 方法获取 PreparedStatement 对象
PreparedStatement 接口是 Statement 的子接口，它表示一条预编译过的 SQL 语句
setXXX() 方法有两个参数，第一个参数是要设置的 SQL 语句中的参数的索引(从 1 开始)，第二个是设置的 SQL 语句中的参数的值

代码的可读性和可维护性. 

PreparedStatement 能最大可能提高性能：
    DBServer会对预编译语句提供性能优化。因为预编译语句有可能被重复调用，
    只要将参数直接传入编译过的语句执行代码中就会得到执行。
    
    在statement语句中,即使是相同操作但因为数据内容不一样,所以整个语句本身不能匹配,没有缓存语句的意义.
    事实是没有数据库会对普通语句编译后的执行代码缓存.这样每执行一次都要对传入的语句编译一次.  

    (语法检查，语义检查，翻译成二进制命令，缓存)

PreparedStatement 可以防止 SQL 注入 


```

```
    /**
     * 执行 SQL 语句, 使用 PreparedStatement
     * @param sql
     * @param args: 填写 SQL 占位符的可变参数
     */
    public static void update(String sql, Object ... args){
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        
        try {
            connection = JDBCTools.getConnection();
            preparedStatement = connection.prepareStatement(sql);
            
            for(int i = 0; i < args.length; i++){
                preparedStatement.setObject(i + 1, args[i]);
            }
            
            preparedStatement.executeUpdate();
            
        } catch (Exception e) {
            e.printStackTrace();
        } finally{
            JDBCTools.releaseDB(null, preparedStatement, connection);
        }
    }
    
```

getCustomer和getSudent方法很类似，下面我们可以写一个通用的方法。
```
    public Customer getCustomer(String sql, Object... args) {
        Customer customer = null;
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        ResultSet resultSet = null;
        try {
            connection = JdbcTools.getConnection();
            preparedStatement = connection.prepareStatement(sql);
            for (int i = 0; i < args.length; i++) {
                preparedStatement.setObject(i + 1, args[i]);
            }
            resultSet = preparedStatement.executeQuery(sql);
            if (resultSet.next()) {
                customer = new Customer(
                        resultSet.getInt(1),
                        resultSet.getString(2),
                        resultSet.getString(3),
                        resultSet.getString(4),
                        resultSet.getString(5),
                        resultSet.getString(6)
                );
            }

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JdbcTools.release(resultSet, preparedStatement, connection);
        }

        return customer;
    }


    public Student getSudent(String sql, Object... args) {
        Student student = null;
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        ResultSet resultSet = null;
        try {
            connection = JdbcTools.getConnection();
            preparedStatement = connection.prepareStatement(sql);
            for (int i = 0; i < args.length; i++) {
                preparedStatement.setObject(i + 1, args[i]);
            }
            resultSet = preparedStatement.executeQuery(sql);
            if (resultSet.next()) {
                student = new Student(resultSet.getInt(1),
                        resultSet.getInt(2),
                        resultSet.getString(3),
                        resultSet.getString(4),
                        resultSet.getString(5),
                        resultSet.getString(6),
                        resultSet.getInt(7)
                );
            }

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JdbcTools.release(resultSet, preparedStatement, connection);
        }

        return student;
    }
```
我们发现代码中间创建这个对象的代码稍微不太一样，不过还是比较类似的，我们可以通过反射来创建对象，通过反射来给对象
对应的属性赋值。
```
if (resultSet.next()) {
    customer = new Customer(
            resultSet.getInt(1),
            resultSet.getString(2),
            resultSet.getString(3),
            resultSet.getString(4),
            resultSet.getString(5),
            resultSet.getString(6)
    );
}

if (resultSet.next()) {
    student = new Student(resultSet.getInt(1),
            resultSet.getInt(2),
            resultSet.getString(3),
            resultSet.getString(4),
            resultSet.getString(5),
            resultSet.getString(6),
            resultSet.getInt(7)
    );
}

```
创建对象比较简单，传入一个Class对象就可以用反射创建对象了。
但是设置属性对应的值麻烦一些，怎么做到属性值和sql相对应呢？
 就是说你查询的sql（`     String sql = "SELECT id id, name my_name, address, phone FROM customers";`）中的id，name，address      等怎么对应到Customer类中的属性id，name呢？？？？
 有时候数据库表中的字段和类中的属性是不一样的 数据库表中有时候会带上下划线的。例如id叫 ct_id,  name是 ct_name等。
 像学生那个表中字段叫id_card    ，学生类里面叫idCard。

>这个时候我们在写sql的时候 列 后面可以写一个别名，这个别名和类中的属性名一致。
>例如：`select flow_id flowID, id_card idCard,  exam_card examCard from student; `

这样就可以把数据库表列名和类中的属性名一一对应上了。然后利用反射设置类中的属性。


ResultSetMetaData
getColumnCount() 获取查询了多少列。
getColumnLabel（）获取指定索引的列的别名，索引是从1开始的。
结合这个2个就可以实现上面说的啦！

```
String sql = "SELECT id id, name my_name, address, phone FROM customers";
resultSet = statement.executeQuery(sql);
ResultSetMetaData resultSetMetaData = resultSet.getMetaData();
for (int i = 0; i < resultSetMetaData.getColumnCount(); i++) {
    //打印每一列的别名
    System.out.println(resultSetMetaData.getColumnLabel(i + 1));
}
```
下面我就可以写出一个通用的获取数据库表，并返回一个相应类的对象的一个通用方法
 `Class<T> clazz,`这个参数就是返回的类的类型
 sql参数
 args参数，是填充sql的值的。
 
```
    @Test
    public void testGet() {
        String sql = null;

        sql = "SELECT flow_id flowId, type, id_card idCard, "
                + "exam_card examCard, student_name studentName, "
                + "location, grade "
                + "FROM examstudent where flow_id = ? ";
        Student student = get(Student.class, sql, 1);
        System.out.println(student);


        sql = "select id,name,address,phone from customers where id = ? ";

        Customer customer = get(Customer.class, sql, 16);
        System.out.println(customer);

    }

输出：
Student{flowId=1, type=4, idCard='410106123123123', examCard='1291728312783', studentName='mamh', location='shanghai', grade=89}
Customer{id=16, name='mamh', address='shanghai', phone='110'}

    public static <T> T get(Class<T> clazz, String sql, Object... args) {
        //获取 id=4 的 customers 数据表的记录, 并打印
        T object = null;

        Connection connection = null;
        PreparedStatement preparedStatement = null;
        ResultSet resultSet = null;

        try {
            connection = JdbcTools.getConnection();
            preparedStatement = connection.prepareStatement(sql);
            for (int i = 0; i < args.length; i++) {
                preparedStatement.setObject(i + 1, args[i]);
            }
            //得到resetset结果集
            resultSet = preparedStatement.executeQuery();
            //从结果集中得到ResultSetMetaData对象，结果集的元数据
            ResultSetMetaData resultSetMetaData = resultSet.getMetaData();
            //创建一个map对象，用来保存结果集
            Map<String, Object> map = new HashMap<>();
            if (resultSet.next()) {
                //处理ResultSetMetaData，填充对应的map，键是数据库表的列的别名，值就是对应的值
                for (int i = 1; i <= resultSetMetaData.getColumnCount(); i++) {
                    String label = resultSetMetaData.getColumnLabel(i);
                    Object value = resultSet.getObject(label);
                    map.put(label, value);
                }
            }
            if (map.size() > 0) {
                //利用反射创建一个对象
                object = clazz.newInstance();
                //遍历map利用反射对对应的属性赋值
                for (Map.Entry<String, Object> entry : map.entrySet()) {
                    String fieldName = entry.getKey();
                    Object fieldValue = entry.getValue();
                    ReflectionUtils.setFieldValue(object, fieldName, fieldValue);
                }
            }
            System.out.println(object);

        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JdbcTools.release(resultSet, preparedStatement, connection);
        }

        return object;
    }
```




----------


其他相关的类

```

public class Student {
    private int flowId;
    private int type;
    private String idCard;
    private String examCard;
    private String studentName;
    private String location;
    private int grade;


    public Student() {
    }

    public Student(int flowId, int type, String idCard, String examCard, String studentName, String location, int grade) {
        this.flowId = flowId;
        this.type = type;
        this.idCard = idCard;
        this.examCard = examCard;
        this.studentName = studentName;
        this.location = location;
        this.grade = grade;
    }

    public int getFlowId() {
        return flowId;
    }

    public void setFlowId(int flowId) {
        this.flowId = flowId;
    }

    public int getType() {
        return type;
    }

    public void setType(int type) {
        this.type = type;
    }

    public String getIdCard() {
        return idCard;
    }

    public void setIdCard(String idCard) {
        this.idCard = idCard;
    }

    public String getExamCard() {
        return examCard;
    }

    public void setExamCard(String examCard) {
        this.examCard = examCard;
    }

    public String getStudentName() {
        return studentName;
    }

    public void setStudentName(String studentName) {
        this.studentName = studentName;
    }

    public String getLocation() {
        return location;
    }

    public void setLocation(String location) {
        this.location = location;
    }

    public int getGrade() {
        return grade;
    }

    public void setGrade(int grade) {
        this.grade = grade;
    }

    @Override
    public String toString() {
        return "Student{" +
                "flowId=" + flowId +
                ", type=" + type +
                ", idCard='" + idCard + '\'' +
                ", examCard='" + examCard + '\'' +
                ", studentName='" + studentName + '\'' +
                ", location='" + location + '\'' +
                ", grade=" + grade +
                '}';
    }


    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        Student student = (Student) o;

        if (flowId != student.flowId) return false;
        if (type != student.type) return false;
        if (grade != student.grade) return false;
        if (idCard != null ? !idCard.equals(student.idCard) : student.idCard != null) return false;
        if (examCard != null ? !examCard.equals(student.examCard) : student.examCard != null) return false;
        if (studentName != null ? !studentName.equals(student.studentName) : student.studentName != null) return false;
        return location != null ? location.equals(student.location) : student.location == null;
    }

    @Override
    public int hashCode() {
        int result = flowId;
        result = 31 * result + type;
        result = 31 * result + (idCard != null ? idCard.hashCode() : 0);
        result = 31 * result + (examCard != null ? examCard.hashCode() : 0);
        result = 31 * result + (studentName != null ? studentName.hashCode() : 0);
        result = 31 * result + (location != null ? location.hashCode() : 0);
        result = 31 * result + grade;
        return result;
    }
}
```
其他相关的类

```
public class Customer implements Serializable {
    private int id;
    private String name;
    private String address;
    private String phone;

    private String cardType;
    private String cardId;


    public Customer() {
    }

    public Customer(Integer id, String name, String address, String phone, String cardType, String cardId) {
        this.id = id;
        this.name = name;
        this.address = address;
        this.phone = phone;
        this.cardType = cardType;
        this.cardId = cardId;
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
其他相关的类

```

/**
 * 反射的 Utils 函数集合
 * 提供访问私有变量, 获取泛型类型 Class, 提取集合中元素属性等 Utils 函数
 *
 * @author Administrator
 */
public class ReflectionUtils {


    /**
     * 通过反射, 获得定义 Class 时声明的父类的泛型参数的类型
     * 如: public EmployeeDao extends BaseDao<Employee, String>
     *
     * @param clazz
     * @param index
     * @return
     */
    @SuppressWarnings("unchecked")
    public static Class getSuperClassGenricType(Class clazz, int index) {
        Type genType = clazz.getGenericSuperclass();

        if (!(genType instanceof ParameterizedType)) {
            return Object.class;
        }

        Type[] params = ((ParameterizedType) genType).getActualTypeArguments();

        if (index >= params.length || index < 0) {
            return Object.class;
        }

        if (!(params[index] instanceof Class)) {
            return Object.class;
        }

        return (Class) params[index];
    }

    /**
     * 通过反射, 获得 Class 定义中声明的父类的泛型参数类型
     * 如: public EmployeeDao extends BaseDao<Employee, String>
     *
     * @param <T>
     * @param clazz
     * @return
     */
    @SuppressWarnings("unchecked")
    public static <T> Class<T> getSuperGenericType(Class clazz) {
        return getSuperClassGenricType(clazz, 0);
    }

    /**
     * 循环向上转型, 获取对象的 DeclaredMethod
     *
     * @param object
     * @param methodName
     * @param parameterTypes
     * @return
     */
    public static Method getDeclaredMethod(Object object, String methodName, Class<?>[] parameterTypes) {

        for (Class<?> superClass = object.getClass(); superClass != Object.class; superClass = superClass.getSuperclass()) {
            try {
                //superClass.getMethod(methodName, parameterTypes);
                return superClass.getDeclaredMethod(methodName, parameterTypes);
            } catch (NoSuchMethodException e) {
                //Method 不在当前类定义, 继续向上转型
            }
            //..
        }

        return null;
    }

    /**
     * 使 filed 变为可访问
     *
     * @param field
     */
    public static void makeAccessible(Field field) {
        if (!Modifier.isPublic(field.getModifiers())) {
            field.setAccessible(true);
        }
    }

    /**
     * 循环向上转型, 获取对象的 DeclaredField
     *
     * @param object
     * @param filedName
     * @return
     */
    public static Field getDeclaredField(Object object, String filedName) {

        for (Class<?> superClass = object.getClass(); superClass != Object.class; superClass = superClass.getSuperclass()) {
            try {
                return superClass.getDeclaredField(filedName);
            } catch (NoSuchFieldException e) {
                //Field 不在当前类定义, 继续向上转型
            }
        }
        return null;
    }

    /**
     * 直接调用对象方法, 而忽略修饰符(private, protected)
     *
     * @param object
     * @param methodName
     * @param parameterTypes
     * @param parameters
     * @return
     * @throws InvocationTargetException
     * @throws IllegalArgumentException
     */
    public static Object invokeMethod(Object object, String methodName, Class<?>[] parameterTypes,
                                      Object[] parameters) throws InvocationTargetException {

        Method method = getDeclaredMethod(object, methodName, parameterTypes);

        if (method == null) {
            throw new IllegalArgumentException("Could not find method [" + methodName + "] on target [" + object + "]");
        }

        method.setAccessible(true);

        try {
            return method.invoke(object, parameters);
        } catch (IllegalAccessException e) {
            System.out.println("不可能抛出的异常");
        }

        return null;
    }

    /**
     * 直接设置对象属性值, 忽略 private/protected 修饰符, 也不经过 setter
     *
     * @param object
     * @param fieldName
     * @param value
     */
    public static void setFieldValue(Object object, String fieldName, Object value) {
        Field field = getDeclaredField(object, fieldName);

        if (field == null)
            throw new IllegalArgumentException("Could not find field [" + fieldName + "] on target [" + object + "]");

        makeAccessible(field);

        try {
            field.set(object, value);
        } catch (IllegalAccessException e) {
            System.out.println("不可能抛出的异常");
        }
    }

    /**
     * 直接读取对象的属性值, 忽略 private/protected 修饰符, 也不经过 getter
     *
     * @param object
     * @param fieldName
     * @return
     */
    public static Object getFieldValue(Object object, String fieldName) {
        Field field = getDeclaredField(object, fieldName);

        if (field == null)
            throw new IllegalArgumentException("Could not find field [" + fieldName + "] on target [" + object + "]");

        makeAccessible(field);

        Object result = null;

        try {
            result = field.get(object);
        } catch (IllegalAccessException e) {
            System.out.println("不可能抛出的异常");
        }

        return result;
    }
}

```


DatabaseMeta描述数据库元数据对象

```
    @Test
    public void testDatabaseMeta() {
        //DatabaseMeta 描述数据库的元数据对象
        Connection connection = null;
        try {
            connection = JdbcTools.getConnection();
            DatabaseMetaData metaData = connection.getMetaData();


            //可以得到数据库本身的一些基本信息
            //数据库版本号
            int databaseMajorVersion = metaData.getDatabaseMajorVersion();
            System.out.println(databaseMajorVersion);

            String userName = metaData.getUserName();
            System.out.println(userName);

            ResultSet catalogs = metaData.getCatalogs();
            while (catalogs.next()){
                System.out.println(catalogs.getString(1));
            }

        } catch (Exception e) {
            e.printStackTrace();
        }


    }
```

获取自动增长的主键的值

```
            connection = JdbcTools.getConnection();
            String sql = "insert into customers(name,address,phone) values(?,?,?)";
            preparedStatement = connection.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS);
            preparedStatement.setObject(1, "abcdefghg");
            preparedStatement.setObject(2, "abc@gamil.com");
            preparedStatement.setObject(3, "sex");

            preparedStatement.executeUpdate();
            
            //获取主键的值
            ResultSet generatedKeys = preparedStatement.getGeneratedKeys();
            while (generatedKeys.next()) {
                System.out.println(generatedKeys.getObject(1));
            }

            //主键列的名称，GENERATED_KEY
            ResultSetMetaData metaData = generatedKeys.getMetaData();
            for (int i = 0; i < metaData.getColumnCount(); i++) {
                System.out.println(metaData.getColumnName(i + 1));
                System.out.println(metaData.getColumnLabel(i + 1));
                System.out.println();

            }
```

如何存储读取blob，大文件
setBlob(4, inputStream)来往数据库里面放入一个大文件
```
        connection = JdbcTools.getConnection();
            String sql = "INSERT INTO customers(name, address, phone, picture) VALUES(?,?,?,?)";
            preparedStatement = connection.prepareStatement(sql);
            preparedStatement.setObject(1, "abc" + new Random().nextInt(10000));
            preparedStatement.setObject(2, "address + " + new Random().nextInt(10000));
            preparedStatement.setObject(3, "phone + " + new Random().nextInt(10000));
            InputStream inputStream = new FileInputStream("/home/mamh/QQ图片20170815153044.jpg");
            //插入一个图片文件
            preparedStatement.setBlob(4, inputStream);
            preparedStatement.executeUpdate();

```


resultSet.getBlob(5)来获取大文件，然后通过inputstream和outputstream 输入输出流来操作
blob.getBinaryStream();来获取一个输入流
```
            connection = JdbcTools.getConnection();
            String sql = "SELECT id,name,address, phone,picture FROM customers WHERE  id = 30";
            preparedStatement = connection.prepareStatement(sql);
            ResultSet resultSet = preparedStatement.executeQuery();
            while (resultSet.next()) {
                System.out.println(resultSet.getObject(1));
                System.out.println(resultSet.getObject(2));
                System.out.println(resultSet.getObject(3));
                System.out.println(resultSet.getObject(4));
                Blob blob = resultSet.getBlob(5);
                InputStream binaryStream = blob.getBinaryStream();
                OutputStream outputStream = new FileOutputStream("/home/mamh/1.jpg");

                byte[] buffer = new byte[1024];
                int len = 0;
                while ((len = binaryStream.read(buffer)) != -1) {
                    outputStream.write(buffer);
                }
                binaryStream.close();
                outputStream.close();
```

数据库的事务操作

```
        //事务操作
        Dao dao = new DaoImpl();
        String sql = null;
        Connection connection = null;//为了保证事务操作，需要这个同一个connection。

        try {
            connection = JdbcTools.getConnection();
            connection.setAutoCommit(false);//设置默认提交为false，不自动提交

            sql = "update users set  balance = (balance - 500) where id = 1";
            dao.update(connection, sql);//这里update方法我们传入一个connection。注意update方法里面就不能再次获取connection了。

            int a = 10 / 0;
            sql = "update users set  balance = (balance + 500) where id = 2";
            dao.update(connection, sql);//这里update方法我们传入一个connection。

            //如果事务操作都成功，则提交事务
            connection.commit();

        } catch (Exception e) {
            try {
                //如果出异常就回滚
                connection.rollback();
            } catch (SQLException e1) {
                e1.printStackTrace();
            }
            e.printStackTrace();
        }
```

数据库的隔离级别
>对于同时运行的多个事务，当这些事务访问数据库中相同的数据时，如果没有采取必要的隔离机制，就会导致各种并发问题：
>>脏读，对于2个事务T1，T2，T1读取了已经被T2更新但还没有被提交的字段。之后若T2回滚，T1读取的内容就是临时且无效的。

>>不可重复读，对于2个事务T1，T2，T1读取了一个字段，然后T2更新了该字段。之后T1再次读取同一个字段，值就不同了。

>>幻读，对于2个事务T1，T2，T1从一个表中读取了一个字段，然后T2在该表中插入了一些新的行，之后，如果T1再次读取同一个表，就会多出几行。

>数据库事务的隔离性：数据库系统必须是具有隔离并发运行各个事务的能力，使他们不会相互影响，避免各种并发问题。
>一个事务与其他事务隔离的程度称为隔离级别，数据库规定了多种隔离级别，不同隔离级别对应不同的干扰程度，隔离级别越高，数据库一致性就越好，但并发性越弱。

>读未提交数据
>读已提交数据
>可重复读
>串行化



批量处理jdbc语句
addBatch()
executeBatch()
clearBatch()

数据库连接池
dbcp数据库连接池，是apache的
c3p0数据库连接池

dbcp的使用
通过创建BasicDataSource的一个实例，然后调用其中的 dataSource.getConnection()方法获得一个Connection对象。
其中dataSource中有一些set方法：
setUsername（）设置数据库访问的帐号
setPassword（）设置密码
setUrl（）设置jdbc的url连接地址
setDriverClassName（）设置jdbc的驱动的类名
```
    @Test
    public void testDbcp() throws SQLException {
        String className = "com.mysql.jdbc.Driver";
        String jdbcUrl = "jdbc:mysql://10.0.63.43/test";
        String user = "test";
        String password = "123456";
        BasicDataSource dataSource = new BasicDataSource();
        dataSource.setUsername(user);
        dataSource.setPassword(password);
        dataSource.setUrl(jdbcUrl);
        dataSource.setDriverClassName(className);

        Connection connection = dataSource.getConnection();
        System.out.println(connection);
    }
```
还有另外更常用的获取dataSource 实例的方式，就是使用BasicDataSourceFactory这个工厂类。
这个类有个createDataSource（）方法来创建dataSource 实例。
调用这个方法时候可以传入一个properties 对象。

```
    @Test
    public void testDbcpFactory() {
        try {
            Properties properties = new Properties();
            InputStream inputStream = Main.class.getClassLoader().getResourceAsStream("dbcp.properties");
            properties.load(inputStream);
            BasicDataSource dataSource = (BasicDataSource) BasicDataSourceFactory.createDataSource(properties);
            System.out.println(dataSource.getMaxWait());

            Connection connection = dataSource.getConnection();
            System.out.println(connection);

        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

首先要有个配置文件dbcp.properties
配置文件中的键名是有要求的，不能随便写的。一般是setXXX方法去掉set，然后首字母变小写。
```
username=test
password=123456
driverClassName=com.mysql.jdbc.Driver
url=jdbc:mysql://10.0.63.43/test
```

c3p0的使用
创建ComboPooledDataSource对象实例，然后调用 set方法，设置帐号，密码等。
最后cpds.getConnection()获得Connection 对象。

```
    @Test
    public void testC3p0() {
        String className = "com.mysql.jdbc.Driver";
        String jdbcUrl = "jdbc:mysql://10.0.63.43/test";
        String user = "test";
        String password = "123456";
        ComboPooledDataSource cpds = new ComboPooledDataSource();
        try {
            cpds.setDriverClass(className);
            cpds.setJdbcUrl(jdbcUrl);
            cpds.setUser(user);
            cpds.setPassword(password);
            Connection connection = cpds.getConnection();
            System.out.println(connection);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```
另外一种更常用的方式是使用配置文件来使用c3p0

```
    @Test
    public void testC3p0() {
        ComboPooledDataSource dataSource = new ComboPooledDataSource("helloc3p0");
        try {
            System.out.println(dataSource.getConnection());
            System.out.println(dataSource.getMaxStatements());
            System.out.println(dataSource.getMaxPoolSize());
        } catch (SQLException e) {
            e.printStackTrace();
        }

    }
```
一般的配置文件常用的格式是xml文件格式
文件名一般是c3p0-config.xml，其中的named-config中的name值要和java代码中
的new ComboPooledDataSource("helloc3p0");中参数值一致。
有了datasource,下面的步骤基本上是一样的，调用getConnection()获得Connection 对象。
```
<?xml version="1.0" encoding="UTF-8"?>
<c3p0-config>

    <named-config name="helloc3p0">

        <!-- 指定连接数据源的基本属性 -->
        <property name="user">atguigu</property>
        <property name="password">123456</property>
        <property name="driverClass">com.mysql.jdbc.Driver</property>
        <property name="jdbcUrl">jdbc:mysql://10.0.63.43/atguigu</property>

        <!-- 若数据库中连接数不足时, 一次向数据库服务器申请多少个连接 -->
        <property name="acquireIncrement">5</property>
        <!-- 初始化数据库连接池时连接的数量 -->
        <property name="initialPoolSize">5</property>
        <!-- 数据库连接池中的最小的数据库连接数 -->
        <property name="minPoolSize">5</property>
        <!-- 数据库连接池中的最大的数据库连接数 -->
        <property name="maxPoolSize">10</property>

        <!-- C3P0 数据库连接池可以维护的 Statement 的个数 -->
        <property name="maxStatements">20</property>
        <!-- 每个连接同时可以使用的 Statement 对象的个数 -->
        <property name="maxStatementsPerConnection">5</property>

    </named-config>

</c3p0-config>

```
使用QueryRunner 来更新数据库，插入，删除，更新等操作
```

    @Test
    public void testQueryRunnerUpdate() {
        QueryRunner queryRunner = new QueryRunner();
        String sql = "delete from customers where id in (?, ?)";
        Connection connection = null;
        try {
            connection = JdbcTools.getConnectionFromPool();
            queryRunner.update(connection, sql, 30, 31);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JdbcTools.release(null, null, connection);
        }
    }
```
使用QueryRunner 查询数据库
调用query（）方法时候传入了一个自定义的类CustomerResultSetHandler，这个类需要实现ResultSetHandler接口
```
    @Test
    public void testQueryRunnerQuery() {
        QueryRunner queryRunner = new QueryRunner();
        Connection connection = null;
        String sql = "select id, name, address,phone from customers ";
        try {
            connection = JdbcTools.getConnectionFromPool();
            List<Customer> customer = queryRunner.query(connection, sql, new CustomerResultSetHandler());
            System.out.println(customer);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JdbcTools.release(null, null, connection);
        }

    }
```
实现ResultSetHandler接口，里面需要实现handle(ResultSet rs)方法，
这个方法的返回值最后就是query（）方法的返回值。在这个handle(ResultSet rs)方法里面
用户可以自己实现处理结果集的代码。
```
public class CustomerResultSetHandler implements ResultSetHandler<List<Customer>> {
    @Override
    public List<Customer> handle(ResultSet rs) throws SQLException {
        System.out.println("handler............");
        List<Customer> list = new ArrayList<>();
        while (rs.next()){
            int id = rs.getInt(1);
            String name = rs.getString(2);
            String address= rs.getString(3);
            String phone= rs.getString(4);
            Customer customer = new Customer(id,name,address,phone);
            list.add(customer);
        }
        return list;
    }
}
```

当然apache组织自己也实现了几个ResultSetHandler接口类，
常用的有ScalarHandler，MapListHandler，BeanMapHandler，MapHandler，BeanListHandler，BeanHandler

```
    @Test
    public void testnScalarHanlder() {
        //把结果集转换为一个数值返回，可以是任意类型，如果结果是多余2列的，只会取得第一列（这里是列啊！）
        QueryRunner queryRunner = new QueryRunner();
        Connection connection = null;
        String sql = "select count(id) from customers ";
        try {
            connection = JdbcTools.getConnectionFromPool();
            Object customer = queryRunner.query(connection, sql, new ScalarHandler<>());
            System.out.println(customer);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JdbcTools.release(null, null, connection);
        }
    }

    @Test
    public void testnMapListHanlder() {
        //将每行转换为一个map，多行转换一个list
        QueryRunner queryRunner = new QueryRunner();
        Connection connection = null;
        String sql = "select id, name, address,phone from customers ";
        try {
            connection = JdbcTools.getConnectionFromPool();
            List<Map<String, Object>> customer = queryRunner.query(connection, sql, new MapListHandler());
            System.out.println(customer);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JdbcTools.release(null, null, connection);
        }
    }

    @Test
    public void testBeanMapHanlder() {
        QueryRunner queryRunner = new QueryRunner();
        Connection connection = null;
        String sql = "select id, name, address,phone from customers ";
        try {
            connection = JdbcTools.getConnectionFromPool();
            Map<String, Customer> customer = queryRunner.query(connection, sql, new BeanMapHandler<String, Customer>(Customer.class));
            System.out.println(customer);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JdbcTools.release(null, null, connection);
        }
    }

    @Test
    public void testMapHanlder() {
        QueryRunner queryRunner = new QueryRunner();
        Connection connection = null;
        String sql = "select id, name, address,phone from customers ";
        try {
            connection = JdbcTools.getConnectionFromPool();
            Map<String, Object> customer = queryRunner.query(connection, sql, new MapHandler());
            System.out.println(customer);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JdbcTools.release(null, null, connection);
        }
    }

    @Test
    public void testBeanListHanlder() {
        QueryRunner queryRunner = new QueryRunner();
        Connection connection = null;
        String sql = "select id, name, address,phone from customers ";

        try {
            connection = JdbcTools.getConnectionFromPool();
            BeanListHandler<Customer> rs = new BeanListHandler<Customer>(Customer.class);
            List<Customer> customer = queryRunner.query(connection, sql, rs);
            System.out.println(customer);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JdbcTools.release(null, null, connection);
        }
    }
```
