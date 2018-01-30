数据库学习oracle之过滤和排序.md

#在查询中过滤行


```sql

SELECT EMPLOYEE_ID,LAST_NAME from EMPLOYEES;


```
使用where来过滤，添加查询条件
```sql

SELECT EMPLOYEE_ID,LAST_NAME,SALARY 
from EMPLOYEES
WHERE EMPLOYEE_ID > 200

```

```sql

SELECT EMPLOYEE_ID,LAST_NAME,SALARY 
from EMPLOYEES
WHERE SALARY > 5000

```
```sql

SELECT EMPLOYEE_ID,LAST_NAME,SALARY 
from EMPLOYEES
WHERE DEPARTMENT_ID > 90

```

字符和日期只能包含在单引号里面。
```sql
SELECT *
FROM EMPLOYEES
WHERE  to_char(HIRE_DATE,'yyyy-mm-dd')='1994-06-07';
```

比较运算符使用的是 “=” 而不是“==”


between ...   and ....
```sql

SELECT *
FROM EMPLOYEES
WHERE  SALARY>=4000 and SALARY<7000;

SELECT *
FROM EMPLOYEES
WHERE SALARY BETWEEN 4000 and 7000;



```


where ...  in ....
```sql

SELECT LAST_NAME,DEPARTMENT_ID
FROM EMPLOYEES
WHERE DEPARTMENT_ID =90 or DEPARTMENT_ID = 70 OR DEPARTMENT_ID = 80;

SELECT LAST_NAME,DEPARTMENT_ID
FROM EMPLOYEES
WHERE DEPARTMENT_ID in (70,80,90);

```

where ... like ....   模糊查询
```sql


SELECT LAST_NAME,DEPARTMENT_ID,SALARY
  FROM EMPLOYEES
WHERE LAST_NAME LIKE '%a%';



```

% 表示匹配多个
_ 下划线表示匹配一个

怎么转义一个字符，escape后面的字符可以自定义的。

```sql


SELECT LAST_NAME,DEPARTMENT_ID,SALARY
  FROM EMPLOYEES
WHERE LAST_NAME LIKE '%\_%' ESCAPE '\';

```

where ... is  null
```sql

SELECT *
  FROM EMPLOYEES
WHERE COMMISSION_PCT is NULL ;

```

where ... is not null

```sql

SELECT *
  FROM EMPLOYEES
WHERE COMMISSION_PCT is NOT NULL ;

```






#在查询中进行排序


order by

--从小到大排序(默认)
```sql

SELECT LAST_NAME,SALARY,DEPARTMENT_ID
  FROM EMPLOYEES
WHERE DEPARTMENT_ID=80 ORDER BY SALARY; 

SELECT LAST_NAME,SALARY,DEPARTMENT_ID
  FROM EMPLOYEES
WHERE DEPARTMENT_ID=80 ORDER BY SALARY ASC ; 
```

使用desc 从大到小排序
```sql

SELECT LAST_NAME,SALARY,DEPARTMENT_ID
  FROM EMPLOYEES
WHERE DEPARTMENT_ID=80 ORDER BY SALARY DESC ;

```

```sql

SELECT LAST_NAME,SALARY,DEPARTMENT_ID
FROM EMPLOYEES
ORDER BY SALARY DESC, DEPARTMENT_ID ASC 

```

多个列排序
```sql

SELECT LAST_NAME,SALARY,DEPARTMENT_ID
FROM EMPLOYEES
ORDER BY SALARY DESC, DEPARTMENT_ID ASC ;

```


#练习题

1.where 子句 紧随 from 子句


2.查询 last name 是 King的员工信息
```sql

SELECT first_name, last_name
FROM EMPLOYEES
WHERE LAST_NAME = 'King'



```


3.查询1998-4-24来公司的员工有哪些

注意 日期必须放在单引号里面

```sql
SELECT LAST_NAME,hire_date
FROM EMPLOYEES
WHERE HIRE_DATE = '24-4月-1998'

--或者这样写： where to_char(hire_date,'yyyy-mm-dd') ='1998-04-24'



```

4.查询工资在5000到10000的员工
```sql

SELECT LAST_NAME,SALARY
FROM EMPLOYEES
WHERE SALARY>=5000  AND  SALARY<=1000;

SELECT LAST_NAME,SALARY
FROM EMPLOYEES
WHERE SALARY BETWEEN 5000 AND  10000;

```


5.查询工资等于6000,7000,8000,9000,10000的员工
```sql

SELECT *
FROM EMPLOYEES
WHERE SALARY=6000 OR SALARY=7000
OR SALARY=8000 OR SALARY=9000
OR SALARY=10000;

SELECT *
FROM EMPLOYEES
WHERE SALARY IN (6000,7000,8000,9000,10000);

```



6.查询last_name 中有 “o” 字符的所有员工信息
```sql

SELECT *
FROM EMPLOYEES
WHERE LAST_NAME LIKE '%o%'


```

7.查询last_name 中 第二个字符是 “o”的所有员工信息
```sql

SELECT *
FROM EMPLOYEES
WHERE LAST_NAME LIKE '_o%'


```

8.查询last_name 中 有 “_” 下划线字符 的所有员工信息
```sql

SELECT *
FROM EMPLOYEES
WHERE LAST_NAME LIKE '%\_%' ESCAPE '\'


```

9.查询commission_pct字段为空的所有员工信息
```sql
SELECT LAST_NAME,commission_pct
FROM EMPLOYEES
WHERE COMMISSION_PCT is NULL ;

```


10.查询commission_pct字段为不是空的所有员工信息
```sql
SELECT LAST_NAME,commission_pct
FROM EMPLOYEES
WHERE COMMISSION_PCT is not NULL ;

```



#测试题


1.查询工资大于12000的员工姓名和工资
```sql

SELECT * FROM EMPLOYEES
WHERE SALARY>12000;
```

2.查询员工号为176的员工的姓名和部门号

```sql
select last_name,department_id from employees
where employee_id=176
```

3.选择工资不在5000到12000的员工的姓名和工资
```
select last_name,salary
from employees
where salary < 5000 or salary > 12000;

--where salary not between 5000 and 120000;

```

4.选择雇用时间在1998-02-01到1998-05-01之间的员工姓名，job_id和雇用时间
```
select last_name,job_id,hire_date
from employees
where to_char(hire_date,'yyyy-mm-dd') between '1998-02-01' and '1998-05-01'
```

5.选择在20或50号部门工作的员工姓名和部门号
```
select last_name,department_id
from employees
where deparment_id in (20,50)
```

6.选择在1994年雇用的员工的姓名和雇用时间
```
select last_name,hire_date
from employees
where to_char(hire_date, 'yyyy') = '1994'
```

7.选择公司中没有管理者的员工姓名及job_id
```
select last_name,job_id
from employees
where manager_id is null
```
8.选择公司中有奖金的员工姓名，工资和奖金级别
```
select last_name,salary,commission_pct
from employees
where commission_pct is not null
```

9.选择员工姓名的第三个字母是a的员工姓名
```
select last_name
from employees
where last_name like '__a%'

```

10.选择姓名中有字母a和e的员工姓名
```
select last_name
from employees
where last_name like '%a%e%' or last_name like '%e%a%'
```




























































































































































































































































































































