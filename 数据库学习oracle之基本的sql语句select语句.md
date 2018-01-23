马哥私房菜的github地址 https://github.com/mageSFC/myblog

数据库学习oracle之基本的sql语句select语句


SQL语句分为以下三种类型：

* DML: Data Manipulation Language 数据操纵语言
* DDL:  Data Definition Language 数据定义语言
* DCL:  Data Control Language 数据控制语言


DML用于查询与修改数据记录，包括如下SQL语句：

* INSERT：添加数据到数据库中
* UPDATE：修改数据库中的数据
* DELETE：删除数据库中的数据
* SELECT：选择（查询）数据


DDL用于定义数据库的结构，比如创建、修改或删除数据库对象，包括如下SQL语句：

* CREATE TABLE：创建数据库表
* ALTER  TABLE：更改表结构、添加、删除、修改列长度
* DROP TABLE：删除表
* CREATE INDEX：在表上建立索引
* DROP INDEX：删除索引


DCL用来控制数据库的访问，包括如下SQL语句：

* GRANT：授予访问权限
* REVOKE：撤销访问权限
* COMMIT：提交事务处理
* ROLLBACK：事务处理回退
* SAVEPOINT：设置保存点
* LOCK：对数据库的特定部分进行锁定


查询定义列的别名：
* 后面直接跟上别名
* 使用as关键字
* 使用as关键字，别名加上双引号，这个时候显示会按照双引号里面的大小写原始显示
```sql

SELECT EMPLOYEE_ID id,LAST_NAME,12*EMPLOYEES.SALARY AS "annual_sal" FROM EMPLOYEE

```

连接符 用 || 表示，用来合成列

类似java中的字符串拼接加号。

```sql

SELECT LAST_NAME || ' de job is :  '|| JOB_ID as details
FROM EMPLOYEES;



```
sql中字符串使用单引号。
日期和字符只能带单引号中出现。只有在别名的时候用的双引号

去除重复的
使用关键字distinct
```sql

SELECT DISTINCT DEPARTMENT_ID FROM  EMPLOYEES;

```

下面是错误的sql语句
```sql

SELECT LAST_NAME, DISTINCT DEPARTMENT_ID FROM  EMPLOYEES;
--ORA-00936: missing expression
```


















