马哥的淘宝店:https://shop592330910.taobao.com/



>DAO data access object
>what：访问数据信息（这个数据不一定在数据库里面存放，也可能是xml文件，txt文件，data文件等）的类，包含了对数据的crud（create，read，update，delete），增删改查的操作。不包含任何逻辑代码。
>
>why：实现功能的模块化，更有利用代码的维护和升级。
>
>how：使用JDBC编写dao

```
    void update(String sql, Object... args) {

    <T> T get(Class<T> clazz, String sql, Object... args) {

    List<T> T getAll(Class<T> clazz, String sql, Object... args) {//这个是查询多条记录，返回一个列表
        马哥的淘宝店:https://shop592330910.taobao.com/
    <E> E get(String sql, Object... args)//返回一个具体的记录的具体的某个列的值
```


```

public class DaoImpl implements Dao {

    static Dao dao;

    static {
        dao = new DaoImpl();
    }
    //特别注意这里不能这样new 马哥的淘宝店:https://shop592330910.taobao.com/
    //Dao dao = new DaoImpl() 在执行下面的测试方法时候 这样会导致递归的创建对象

    @Test
    public void testUpdate() {
        String sql = "insert into customers(name,address,phone) values(?,?,?)";
        update(sql, "tom", "beijing", "123456123");
    }


    @Override
    public void update(String sql, Object... args) {
        Connection connection = null;
        PreparedStatement preparedStatement = null;
        try {                       马哥的淘宝店:https://shop592330910.taobao.com/
            connection = JdbcTools.getConnection();
            preparedStatement = connection.prepareStatement(sql);
            for (int i = 0; i < args.length; i++) {
                preparedStatement.setObject(i + 1, args[i]);
            }
            preparedStatement.executeUpdate();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            JdbcTools.release(null, preparedStatement, connection);
        }
    }

    @Override
    public <T> T get(Class<T> clazz, String sql, Object... args) {
        //获取 id=4 的 customers 数据表的记录, 并打印
        T object = null; 马哥的淘宝店:https://shop592330910.taobao.com/

        Connection connection = null;
        PreparedStatement preparedStatement = null;
        ResultSet resultSet = null;
        ResultSetMetaData resultSetMetaData = null;
        try {
            connection = JdbcTools.getConnection();
            preparedStatement = connection.prepareStatement(sql);
            for (int i = 0; i < args.length; i++) {
                preparedStatement.setObject(i + 1, args[i]);
            }
            resultSet = preparedStatement.executeQuery();
            resultSetMetaData = resultSet.getMetaData();

            if (resultSet.next()) {
                object = clazz.newInstance();
                for (int i = 1; i <= resultSetMetaData.getColumnCount(); i++) {
                    String fieldName = resultSetMetaData.getColumnLabel(i);
                    Object fieldValue = resultSet.getObject(fieldName);
                   
                    //ReflectionUtils.setFieldValue(object, fieldName, fieldValue);

                    BeanUtils.setProperty(object, fieldName, fieldValue);

                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            //6. 关闭数据库资源.  马哥的淘宝店:https://shop592330910.taobao.com/
            JdbcTools.release(resultSet, preparedStatement, connection);
        }

        return object;
    }

```

javaEE中，java类的属性通过getter，setter来定义，get或set方法去掉get或set后那个名称首字母变小写。即为java类的属性。
以前叫的那个叫属性，即成员变量，称之为字段
操作java 类属性的一个工具包 beanutils。

其中有2个方法
setProperty()
getProperty()

```
上面的代码用的反射来给属性赋值的，专业的应该用beanutils工具类的方法
马哥的淘宝店:https://shop592330910.taobao.com/
BeanUtils.setProperty(object, fieldName, fieldValue);
```


