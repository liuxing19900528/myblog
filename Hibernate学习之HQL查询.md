

Hibernate学习之HQL查询

HQL(hibernate query language)是面向对象的查询语言，他和sql查询语言有些相似。
在hibernate提供的各种检索方式中，hql是使用最广泛的一种检索方式，它具有如下功能：
1.在查询语句中设定各种查询条件
2.支持投影查询，即仅检索出对象的部分属性
3.支持分页查询
4.支持链接查询
5.支持分组查询，允许使用having和group by关键字
6.提供内置聚集函数，如sum(),min(),max()
7.支持子查询
8.支持动态绑定参数
9.能够调用用户定义的sql函数或者标准的sql函数

下面的例子我们采用了oracle数据库
oracle数据库安装在了windows上面。

oracle远程访问开启：

解决方法：

查看端口状态：CMD -> netstat -a -n
显示的结果是：1521端口对应的本地地址栏为：127.0.0.0:1521,
此时，修改Oracle安装目录下dbhome_1\NETWORK\ADMIN\listener.ora文件（或是PLSQL下的对应文件instantclient_11_2\listener.ora）的HOST值，改为0.0.0.0。
重启Oracle监听服务；
再次通过netstat查看端口信息,显示：1521端口对应的本地地址栏为：0.0.0.0:1521
这时通过127.0.0.1或loaclhost或主机名或本机ip都可telnet通过。

 在Windows系统下完成Oracle安装后，在其防火墙设置中开放1521端口（Oracle默认的侦听端口）。若客户端仍然无法访问，则需要作进一步的设置，即在注册表“HKEY_LOCAL_MACHINE” - "Software" - “ORACLE” - "HOME"下添加一个注册表项“USE_SHARED_SOCKED”，并将其值设为TRUE，然后重启Oracle服务及Listener服务。


总结：
 Oracle Telnet 1521失败，要检查以下几点：
1、防火墙是否开启，若开启，是否有对1521端口开启；
2、listener.ora文件的HOST值。

注：10.2以上，USE_SHARDED_SOCKET就已经是默认值为TRUE了，无需再修改。

最后通过nmap命令可以看到1521端口是open的状态表明可以了
```
$ nmap 10.0.63.42                                                                                                                           

Starting Nmap 7.01 ( https://nmap.org ) at 2017-12-22 16:01 CST
Nmap scan report for 10.0.63.42
Host is up (0.88s latency).
Not shown: 987 closed ports
PORT      STATE SERVICE
1521/tcp  open  oracle


Nmap done: 1 IP address (1 host up) scanned in 5.13 seconds
```

添加oracle的jdbc驱动文件
```
把从官网下载的oracle的jdbc驱动jar包放到家目录下面执行下面命令：
$ mvn install:install-file -Dfile=ojdbc8.jar -DgroupId=com.oracle -DartifactId=ojdbc8 -Dversion=12 -Dpackaging=jar                          [mamh@10.0.63.43 ] 17-12-22 15:53  /home/mamh
[INFO] Scanning for projects...
[INFO]                                                                         
[INFO] ------------------------------------------------------------------------
[INFO] Building Maven Stub Project (No POM) 1
[INFO] ------------------------------------------------------------------------
[INFO] 
[INFO] --- maven-install-plugin:2.5.2:install-file (default-cli) @ standalone-pom ---
[INFO] pom.xml not found in ojdbc8.jar
[INFO] Installing /home/mamh/ojdbc8.jar to /home/mamh/.m2/repository/com/oracle/ojdbc8/12/ojdbc8-12.jar
[INFO] Installing /tmp/mvninstall5920371845117601504.pom to /home/mamh/.m2/repository/com/oracle/ojdbc8/12/ojdbc8-12.pom
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time: 0.332 s
[INFO] Finished at: 2017-12-22T15:54:02+08:00
[INFO] Final Memory: 8M/303M
[INFO] ------------------------------------------------------------------------
$ cat /home/mamh/.m2/repository/com/oracle/ojdbc8/12/ojdbc8-12.pom                                                                          [mamh@10.0.63.43 ] 17-12-22 15:54  /home/mamh
<?xml version="1.0" encoding="UTF-8"?>
<project xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd" xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <modelVersion>4.0.0</modelVersion>
  <groupId>com.oracle</groupId>
  <artifactId>ojdbc8</artifactId>
  <version>12</version>
  <description>POM was created from install:install-file</description>
</project>

最后pom.xml中添加如下：
<dependency>
    <groupId>com.oracle</groupId>
    <artifactId>ojdbc8</artifactId>
    <version>12</version>
</dependency>



```



----------


这里我们给出一个测试使用的数据库表，包含数据的


|#|EMPLOYEE_ID|FIRST_NAME|LAST_NAME|EMAIL|PHONE_NUMBER|HIRE_DATE|JOB_ID|SALARY|COMMISSION_PCT|MANAGER_ID|DEPARTMENT_ID|
|-|-|-|-|-|-|-|-|-|-|-|-|
|1|100|Steven|King|SKING|515.123.4567|1987-06-17 00:00:00|AD_PRES|24000.00|NULL|NULL|90|
|2|101|Neena|Kochhar|NKOCHHAR|515.123.4568|1989-09-21 00:00:00|AD_VP|17000.00|NULL|100|90|
|3|102|Lex|De Haan|LDEHAAN|515.123.4569|1993-01-13 00:00:00|AD_VP|17000.00|NULL|100|90|
|4|103|Alexander|Hunold|AHUNOLD|590.423.4567|1990-01-03 00:00:00|IT_PROG|9000.00|NULL|102|60|
|5|104|Bruce|Ernst|BERNST|590.423.4568|1991-05-21 00:00:00|IT_PROG|6000.00|NULL|103|60|
|6|105|David|Austin|DAUSTIN|590.423.4569|1997-06-25 00:00:00|IT_PROG|4800.00|NULL|103|60|
|7|106|Valli|Pataballa|VPATABAL|590.423.4560|1998-02-05 00:00:00|IT_PROG|4800.00|NULL|103|60|
|8|107|Diana|Lorentz|DLORENTZ|590.423.5567|1999-02-07 00:00:00|IT_PROG|4200.00|NULL|103|60|
|9|108|Nancy|Greenberg|NGREENBE|515.124.4569|1994-08-17 00:00:00|FI_MGR|12000.00|NULL|101|100|
|10|109|Daniel|Faviet|DFAVIET|515.124.4169|1994-08-16 00:00:00|FI_ACCOUNT|9000.00|NULL|108|100|
|11|110|John|Chen|JCHEN|515.124.4269|1997-09-28 00:00:00|FI_ACCOUNT|8200.00|NULL|108|100|
|12|111|Ismael|Sciarra|ISCIARRA|515.124.4369|1997-09-30 00:00:00|FI_ACCOUNT|7700.00|NULL|108|100|
|13|112|Jose Manuel|Urman|JMURMAN|515.124.4469|1998-03-07 00:00:00|FI_ACCOUNT|7800.00|NULL|108|100|
|14|113|Luis|Popp|LPOPP|515.124.4567|1999-12-07 00:00:00|FI_ACCOUNT|6900.00|NULL|108|100|
|15|114|Den|Raphaely|DRAPHEAL|515.127.4561|1994-12-07 00:00:00|PU_MAN|11000.00|NULL|100|30|
|16|115|Alexander|Khoo|AKHOO|515.127.4562|1995-05-18 00:00:00|PU_CLERK|3100.00|NULL|114|30|
|17|116|Shelli|Baida|SBAIDA|515.127.4563|1997-12-24 00:00:00|PU_CLERK|2900.00|NULL|114|30|
|18|117|Sigal|Tobias|STOBIAS|515.127.4564|1997-07-24 00:00:00|PU_CLERK|2800.00|NULL|114|30|
|19|118|Guy|Himuro|GHIMURO|515.127.4565|1998-11-15 00:00:00|PU_CLERK|2600.00|NULL|114|30|
|20|119|Karen|Colmenares|KCOLMENA|515.127.4566|1999-08-10 00:00:00|PU_CLERK|2500.00|NULL|114|30|
|21|120|Matthew|Weiss|MWEISS|650.123.1234|1996-07-18 00:00:00|ST_MAN|8000.00|NULL|100|50|
|22|121|Adam|Fripp|AFRIPP|650.123.2234|1997-04-10 00:00:00|ST_MAN|8200.00|NULL|100|50|
|23|122|Payam|Kaufling|PKAUFLIN|650.123.3234|1995-05-01 00:00:00|ST_MAN|7900.00|NULL|100|50|
|24|123|Shanta|Vollman|SVOLLMAN|650.123.4234|1997-10-10 00:00:00|ST_MAN|6500.00|NULL|100|50|
|25|124|Kevin|Mourgos|KMOURGOS|650.123.5234|1999-11-16 00:00:00|ST_MAN|5800.00|NULL|100|50|
|26|125|Julia|Nayer|JNAYER|650.124.1214|1997-07-16 00:00:00|ST_CLERK|3200.00|NULL|120|50|
|27|126|Irene|Mikkilineni|IMIKKILI|650.124.1224|1998-09-28 00:00:00|ST_CLERK|2700.00|NULL|120|50|
|28|127|James|Landry|JLANDRY|650.124.1334|1999-01-14 00:00:00|ST_CLERK|2400.00|NULL|120|50|
|29|128|Steven|Markle|SMARKLE|650.124.1434|2000-03-08 00:00:00|ST_CLERK|2200.00|NULL|120|50|
|30|129|Laura|Bissot|LBISSOT|650.124.5234|1997-08-20 00:00:00|ST_CLERK|3300.00|NULL|121|50|
|31|130|Mozhe|Atkinson|MATKINSO|650.124.6234|1997-10-30 00:00:00|ST_CLERK|2800.00|NULL|121|50|
|32|131|James|Marlow|JAMRLOW|650.124.7234|1997-02-16 00:00:00|ST_CLERK|2500.00|NULL|121|50|
|33|132|TJ|Olson|TJOLSON|650.124.8234|1999-04-10 00:00:00|ST_CLERK|2100.00|NULL|121|50|
|34|133|Jason|Mallin|JMALLIN|650.127.1934|1996-06-14 00:00:00|ST_CLERK|3300.00|NULL|122|50|
|35|134|Michael|Rogers|MROGERS|650.127.1834|1998-08-26 00:00:00|ST_CLERK|2900.00|NULL|122|50|
|36|135|Ki|Gee|KGEE|650.127.1734|1999-12-12 00:00:00|ST_CLERK|2400.00|NULL|122|50|
|37|136|Hazel|Philtanker|HPHILTAN|650.127.1634|2000-02-06 00:00:00|ST_CLERK|2200.00|NULL|122|50|
|38|137|Renske|Ladwig|RLADWIG|650.121.1234|1995-07-14 00:00:00|ST_CLERK|3600.00|NULL|123|50|
|39|138|Stephen|Stiles|SSTILES|650.121.2034|1997-10-26 00:00:00|ST_CLERK|3200.00|NULL|123|50|
|40|139|John|Seo|JSEO|650.121.2019|1998-02-12 00:00:00|ST_CLERK|2700.00|NULL|123|50|
|41|140|Joshua|Patel|JPATEL|650.121.1834|1998-04-06 00:00:00|ST_CLERK|2500.00|NULL|123|50|
|42|141|Trenna|Rajs|TRAJS|650.121.8009|1995-10-17 00:00:00|ST_CLERK|3500.00|NULL|124|50|
|43|142|Curtis|Davies|CDAVIES|650.121.2994|1997-01-29 00:00:00|ST_CLERK|3100.00|NULL|124|50|
|44|143|Randall|Matos|RMATOS|650.121.2874|1998-03-15 00:00:00|ST_CLERK|2600.00|NULL|124|50|
|45|144|Peter|Vargas|PVARGAS|650.121.2004|1998-07-09 00:00:00|ST_CLERK|2500.00|NULL|124|50|
|46|145|John|Russell|JRUSSEL|011.44.1344.429268|1996-10-01 00:00:00|SA_MAN|14000.00|0.40|100|80|
|47|146|Karen|Partners|KPARTNER|011.44.1344.467268|1997-01-05 00:00:00|SA_MAN|13500.00|0.30|100|80|
|48|147|Alberto|Errazuriz|AERRAZUR|011.44.1344.429278|1997-03-10 00:00:00|SA_MAN|12000.00|0.30|100|80|
|49|148|Gerald|Cambrault|GCAMBRAU|011.44.1344.619268|1999-10-15 00:00:00|SA_MAN|11000.00|0.30|100|80|
|50|149|Eleni|Zlotkey|EZLOTKEY|011.44.1344.429018|2000-01-29 00:00:00|SA_MAN|10500.00|0.20|100|80|
|51|150|Peter|Tucker|PTUCKER|011.44.1344.129268|1997-01-30 00:00:00|SA_REP|10000.00|0.30|145|80|
|52|151|David|Bernstein|DBERNSTE|011.44.1344.345268|1997-03-24 00:00:00|SA_REP|9500.00|0.25|145|80|
|53|152|Peter|Hall|PHALL|011.44.1344.478968|1997-08-20 00:00:00|SA_REP|9000.00|0.25|145|80|
|54|153|Christopher|Olsen|COLSEN|011.44.1344.498718|1998-03-30 00:00:00|SA_REP|8000.00|0.20|145|80|
|55|154|Nanette|Cambrault|NCAMBRAU|011.44.1344.987668|1998-12-09 00:00:00|SA_REP|7500.00|0.20|145|80|
|56|155|Oliver|Tuvault|OTUVAULT|011.44.1344.486508|1999-11-23 00:00:00|SA_REP|7000.00|0.15|145|80|
|57|156|Janette|King|JKING|011.44.1345.429268|1996-01-30 00:00:00|SA_REP|10000.00|0.35|146|80|
|58|157|Patrick|Sully|PSULLY|011.44.1345.929268|1996-03-04 00:00:00|SA_REP|9500.00|0.35|146|80|
|59|158|Allan|McEwen|AMCEWEN|011.44.1345.829268|1996-08-01 00:00:00|SA_REP|9000.00|0.35|146|80|
|60|159|Lindsey|Smith|LSMITH|011.44.1345.729268|1997-03-10 00:00:00|SA_REP|8000.00|0.30|146|80|
|61|160|Louise|Doran|LDORAN|011.44.1345.629268|1997-12-15 00:00:00|SA_REP|7500.00|0.30|146|80|
|62|161|Sarath|Sewall|SSEWALL|011.44.1345.529268|1998-11-03 00:00:00|SA_REP|7000.00|0.25|146|80|
|63|162|Clara|Vishney|CVISHNEY|011.44.1346.129268|1997-11-11 00:00:00|SA_REP|10500.00|0.25|147|80|
|64|163|Danielle|Greene|DGREENE|011.44.1346.229268|1999-03-19 00:00:00|SA_REP|9500.00|0.15|147|80|
|65|164|Mattea|Marvins|MMARVINS|011.44.1346.329268|2000-01-24 00:00:00|SA_REP|7200.00|0.10|147|80|
|66|165|David|Lee|DLEE|011.44.1346.529268|2000-02-23 00:00:00|SA_REP|6800.00|0.10|147|80|
|67|166|Sundar|Ande|SANDE|011.44.1346.629268|2000-03-24 00:00:00|SA_REP|6400.00|0.10|147|80|
|68|167|Amit|Banda|ABANDA|011.44.1346.729268|2000-04-21 00:00:00|SA_REP|6200.00|0.10|147|80|
|69|168|Lisa|Ozer|LOZER|011.44.1343.929268|1997-03-11 00:00:00|SA_REP|11500.00|0.25|148|80|
|70|169|Harrison|Bloom|HBLOOM|011.44.1343.829268|1998-03-23 00:00:00|SA_REP|10000.00|0.20|148|80|
|71|170|Tayler|Fox|TFOX|011.44.1343.729268|1998-01-24 00:00:00|SA_REP|9600.00|0.20|148|80|
|72|171|William|Smith|WSMITH|011.44.1343.629268|1999-02-23 00:00:00|SA_REP|7400.00|0.15|148|80|
|73|172|Elizabeth|Bates|EBATES|011.44.1343.529268|1999-03-24 00:00:00|SA_REP|7300.00|0.15|148|80|
|74|173|Sundita|Kumar|SKUMAR|011.44.1343.329268|2000-04-21 00:00:00|SA_REP|6100.00|0.10|148|80|
|75|174|Ellen|Abel|EABEL|011.44.1644.429267|1996-05-11 00:00:00|SA_REP|11000.00|0.30|149|80|
|76|175|Alyssa|Hutton|AHUTTON|011.44.1644.429266|1997-03-19 00:00:00|SA_REP|8800.00|0.25|149|80|
|77|176|Jonathon|Taylor|JTAYLOR|011.44.1644.429265|1998-03-24 00:00:00|SA_REP|8600.00|0.20|149|80|
|78|177|Jack|Livingston|JLIVINGS|011.44.1644.429264|1998-04-23 00:00:00|SA_REP|8400.00|0.20|149|80|
|79|178|Kimberely|Grant|KGRANT|011.44.1644.429263|1999-05-24 00:00:00|SA_REP|7000.00|0.15|149|NULL|
|80|179|Charles|Johnson|CJOHNSON|011.44.1644.429262|2000-01-04 00:00:00|SA_REP|6200.00|0.10|149|80|
|81|180|Winston|Taylor|WTAYLOR|650.507.9876|1998-01-24 00:00:00|SH_CLERK|3200.00|NULL|120|50|
|82|181|Jean|Fleaur|JFLEAUR|650.507.9877|1998-02-23 00:00:00|SH_CLERK|3100.00|NULL|120|50|
|83|182|Martha|Sullivan|MSULLIVA|650.507.9878|1999-06-21 00:00:00|SH_CLERK|2500.00|NULL|120|50|
|84|183|Girard|Geoni|GGEONI|650.507.9879|2000-02-03 00:00:00|SH_CLERK|2800.00|NULL|120|50|
|85|184|Nandita|Sarchand|NSARCHAN|650.509.1876|1996-01-27 00:00:00|SH_CLERK|4200.00|NULL|121|50|
|86|185|Alexis|Bull|ABULL|650.509.2876|1997-02-20 00:00:00|SH_CLERK|4100.00|NULL|121|50|
|87|186|Julia|Dellinger|JDELLING|650.509.3876|1998-06-24 00:00:00|SH_CLERK|3400.00|NULL|121|50|
|88|187|Anthony|Cabrio|ACABRIO|650.509.4876|1999-02-07 00:00:00|SH_CLERK|3000.00|NULL|121|50|
|89|188|Kelly|Chung|KCHUNG|650.505.1876|1997-06-14 00:00:00|SH_CLERK|3800.00|NULL|122|50|
|90|189|Jennifer|Dilly|JDILLY|650.505.2876|1997-08-13 00:00:00|SH_CLERK|3600.00|NULL|122|50|
|91|190|Timothy|Gates|TGATES|650.505.3876|1998-07-11 00:00:00|SH_CLERK|2900.00|NULL|122|50|
|92|191|Randall|Perkins|RPERKINS|650.505.4876|1999-12-19 00:00:00|SH_CLERK|2500.00|NULL|122|50|
|93|192|Sarah|Bell|SBELL|650.501.1876|1996-02-04 00:00:00|SH_CLERK|4000.00|NULL|123|50|
|94|193|Britney|Everett|BEVERETT|650.501.2876|1997-03-03 00:00:00|SH_CLERK|3900.00|NULL|123|50|
|95|194|Samuel|McCain|SMCCAIN|650.501.3876|1998-07-01 00:00:00|SH_CLERK|3200.00|NULL|123|50|
|96|195|Vance|Jones|VJONES|650.501.4876|1999-03-17 00:00:00|SH_CLERK|2800.00|NULL|123|50|
|97|196|Alana|Walsh|AWALSH|650.507.9811|1998-04-24 00:00:00|SH_CLERK|3100.00|NULL|124|50|
|98|197|Kevin|Feeney|KFEENEY|650.507.9822|1998-05-23 00:00:00|SH_CLERK|3000.00|NULL|124|50|
|99|198|Donald|OConnell|DOCONNEL|650.507.9833|1999-06-21 00:00:00|SH_CLERK|2600.00|NULL|124|50|
|100|199|Douglas|Grant|DGRANT|650.507.9844|2000-01-13 00:00:00|SH_CLERK|2600.00|NULL|124|50|
|101|200|Jennifer|Whalen|JWHALEN|515.123.4444|1987-09-17 00:00:00|AD_ASST|4400.00|NULL|101|10|
|102|201|Michael|Hartstein|MHARTSTE|515.123.5555|1996-02-17 00:00:00|MK_MAN|13000.00|NULL|100|20|
|103|202|Pat|Fay|PFAY|603.123.6666|1997-08-17 00:00:00|MK_REP|6000.00|NULL|201|20|
|104|203|Susan|Mavris|SMAVRIS|515.123.7777|1994-06-07 00:00:00|HR_REP|6500.00|NULL|101|40|
|105|204|Hermann|Baer|HBAER|515.123.8888|1994-06-07 00:00:00|PR_REP|10000.00|NULL|101|70|
|106|205|Shelley|Higgins|SHIGGINS|515.123.8080|1994-06-07 00:00:00|AC_MGR|12000.00|NULL|101|110|
|107|206|William|Gietz|WGIETZ|515.123.8181|1994-06-07 00:00:00|AC_ACCOUNT|8300.00|NULL|205|110|


----------


DEPARTMENT表格
|#|DEPARTMENT_ID|DEPARTMENT_NAME|MANAGER_ID|LOCATION_ID|
|-|-|-|-|-|
|1|10|Administration|200|1700|
|2|20|Marketing|201|1800|
|3|30|Purchasing|114|1700|
|4|40|Human Resources|203|2400|
|5|50|Shipping|121|1500|
|6|60|IT|103|1400|
|7|70|Public Relations|204|2700|
|8|80|Sales|145|2500|
|9|90|Executive|100|1700|
|10|100|Finance|108|1700|
|11|110|Accounting|205|1700|
|12|120|Treasury|NULL|1700|
|13|130|Corporate Tax|NULL|1700|
|14|140|Control And Credit|NULL|1700|
|15|150|Shareholder Services|NULL|1700|
|16|160|Benefits|NULL|1700|
|17|170|Manufacturing|NULL|1700|
|18|180|Construction|NULL|1700|
|19|190|Contracting|NULL|1700|
|20|200|Operations|NULL|1700|
|21|210|IT Support|NULL|1700|
|22|220|NOC|NULL|1700|
|23|230|IT Helpdesk|NULL|1700|
|24|240|Government Sales|NULL|1700|
|25|250|Retail Sales|NULL|1700|
|26|260|Recruiting|NULL|1700|
|27|270|Payroll|NULL|1700|


----------


```

SET VERIFY OFF
ALTER SESSION SET NLS_LANGUAGE=American; 

REM ***************************insert data into the REGIONS table

Prompt ******  Populating REGIONS table ....

INSERT INTO regions VALUES 
        ( 1
        , 'Europe' 
        );

INSERT INTO regions VALUES 
        ( 2
        , 'Americas' 
        );

INSERT INTO regions VALUES 
        ( 3
        , 'Asia' 
        );

INSERT INTO regions VALUES 
        ( 4
        , 'Middle East and Africa' 
        );

REM ***************************insert data into the COUNTRIES table

Prompt ******  Populating COUNTIRES table ....

INSERT INTO countries VALUES 
        ( 'IT'
        , 'Italy'
        , 1 
        );

INSERT INTO countries VALUES 
        ( 'JP'
        , 'Japan'
	, 3 
        );

INSERT INTO countries VALUES 
        ( 'US'
        , 'United States of America'
        , 2 
        );

INSERT INTO countries VALUES 
        ( 'CA'
        , 'Canada'
        , 2 
        );

INSERT INTO countries VALUES 
        ( 'CN'
        , 'China'
        , 3 
        );

INSERT INTO countries VALUES 
        ( 'IN'
        , 'India'
        , 3 
        );

INSERT INTO countries VALUES 
        ( 'AU'
        , 'Australia'
        , 3 
        );

INSERT INTO countries VALUES 
        ( 'ZW'
        , 'Zimbabwe'
        , 4 
        );

INSERT INTO countries VALUES 
        ( 'SG'
        , 'Singapore'
        , 3 
        );

INSERT INTO countries VALUES 
        ( 'UK'
        , 'United Kingdom'
        , 1 
        );

INSERT INTO countries VALUES 
        ( 'FR'
        , 'France'
        , 1 
        );

INSERT INTO countries VALUES 
        ( 'DE'
        , 'Germany'
        , 1 
        );

INSERT INTO countries VALUES 
        ( 'ZM'
        , 'Zambia'
        , 4 
        );

INSERT INTO countries VALUES 
        ( 'EG'
        , 'Egypt'
        , 4 
        );

INSERT INTO countries VALUES 
        ( 'BR'
        , 'Brazil'
        , 2 
        );

INSERT INTO countries VALUES 
        ( 'CH'
        , 'Switzerland'
        , 1 
        );

INSERT INTO countries VALUES 
        ( 'NL'
        , 'Netherlands'
        , 1 
        );

INSERT INTO countries VALUES 
        ( 'MX'
        , 'Mexico'
        , 2 
        );

INSERT INTO countries VALUES 
        ( 'KW'
        , 'Kuwait'
        , 4 
        );

INSERT INTO countries VALUES 
        ( 'IL'
        , 'Israel'
        , 4 
        );

INSERT INTO countries VALUES 
        ( 'DK'
        , 'Denmark'
        , 1 
        );

INSERT INTO countries VALUES 
        ( 'HK'
        , 'HongKong'
        , 3 
        );

INSERT INTO countries VALUES 
        ( 'NG'
        , 'Nigeria'
        , 4 
        );

INSERT INTO countries VALUES 
        ( 'AR'
        , 'Argentina'
        , 2 
        );

INSERT INTO countries VALUES 
        ( 'BE'
        , 'Belgium'
        , 1 
        );


REM ***************************insert data into the LOCATIONS table

Prompt ******  Populating LOCATIONS table ....

INSERT INTO locations VALUES 
        ( 1000 
        , '1297 Via Cola di Rie'
        , '00989'
        , 'Roma'
        , NULL
        , 'IT'
        );

INSERT INTO locations VALUES 
        ( 1100 
        , '93091 Calle della Testa'
        , '10934'
        , 'Venice'
        , NULL
        , 'IT'
        );

INSERT INTO locations VALUES 
        ( 1200 
        , '2017 Shinjuku-ku'
        , '1689'
        , 'Tokyo'
        , 'Tokyo Prefecture'
        , 'JP'
        );

INSERT INTO locations VALUES 
        ( 1300 
        , '9450 Kamiya-cho'
        , '6823'
        , 'Hiroshima'
        , NULL
        , 'JP'
        );

INSERT INTO locations VALUES 
        ( 1400 
        , '2014 Jabberwocky Rd'
        , '26192'
        , 'Southlake'
        , 'Texas'
        , 'US'
        );

INSERT INTO locations VALUES 
        ( 1500 
        , '2011 Interiors Blvd'
        , '99236'
        , 'South San Francisco'
        , 'California'
        , 'US'
        );

INSERT INTO locations VALUES 
        ( 1600 
        , '2007 Zagora St'
        , '50090'
        , 'South Brunswick'
        , 'New Jersey'
        , 'US'
        );

INSERT INTO locations VALUES 
        ( 1700 
        , '2004 Charade Rd'
        , '98199'
        , 'Seattle'
        , 'Washington'
        , 'US'
        );

INSERT INTO locations VALUES 
        ( 1800 
        , '147 Spadina Ave'
        , 'M5V 2L7'
        , 'Toronto'
        , 'Ontario'
        , 'CA'
        );

INSERT INTO locations VALUES 
        ( 1900 
        , '6092 Boxwood St'
        , 'YSW 9T2'
        , 'Whitehorse'
        , 'Yukon'
        , 'CA'
        );

INSERT INTO locations VALUES 
        ( 2000 
        , '40-5-12 Laogianggen'
        , '190518'
        , 'Beijing'
        , NULL
        , 'CN'
        );

INSERT INTO locations VALUES 
        ( 2100 
        , '1298 Vileparle (E)'
        , '490231'
        , 'Bombay'
        , 'Maharashtra'
        , 'IN'
        );

INSERT INTO locations VALUES 
        ( 2200 
        , '12-98 Victoria Street'
        , '2901'
        , 'Sydney'
        , 'New South Wales'
        , 'AU'
        );

INSERT INTO locations VALUES 
        ( 2300 
        , '198 Clementi North'
        , '540198'
        , 'Singapore'
        , NULL
        , 'SG'
        );

INSERT INTO locations VALUES 
        ( 2400 
        , '8204 Arthur St'
        , NULL
        , 'London'
        , NULL
        , 'UK'
        );

INSERT INTO locations VALUES 
        ( 2500 
        , 'Magdalen Centre, The Oxford Science Park'
        , 'OX9 9ZB'
        , 'Oxford'
        , 'Oxford'
        , 'UK'
        );

INSERT INTO locations VALUES 
        ( 2600 
        , '9702 Chester Road'
        , '09629850293'
        , 'Stretford'
        , 'Manchester'
        , 'UK'
        );

INSERT INTO locations VALUES 
        ( 2700 
        , 'Schwanthalerstr. 7031'
        , '80925'
        , 'Munich'
        , 'Bavaria'
        , 'DE'
        );

INSERT INTO locations VALUES 
        ( 2800 
        , 'Rua Frei Caneca 1360 '
        , '01307-002'
        , 'Sao Paulo'
        , 'Sao Paulo'
        , 'BR'
        );

INSERT INTO locations VALUES 
        ( 2900 
        , '20 Rue des Corps-Saints'
        , '1730'
        , 'Geneva'
        , 'Geneve'
        , 'CH'
        );

INSERT INTO locations VALUES 
        ( 3000 
        , 'Murtenstrasse 921'
        , '3095'
        , 'Bern'
        , 'BE'
        , 'CH'
        );

INSERT INTO locations VALUES 
        ( 3100 
        , 'Pieter Breughelstraat 837'
        , '3029SK'
        , 'Utrecht'
        , 'Utrecht'
        , 'NL'
        );

INSERT INTO locations VALUES 
        ( 3200 
        , 'Mariano Escobedo 9991'
        , '11932'
        , 'Mexico City'
        , 'Distrito Federal,'
        , 'MX'
        );


REM ****************************insert data into the DEPARTMENTS table

Prompt ******  Populating DEPARTMENTS table ....

REM disable integrity constraint to EMPLOYEES to load data

ALTER TABLE departments 
  DISABLE CONSTRAINT dept_mgr_fk;

INSERT INTO departments VALUES 
        ( 10
        , 'Administration'
        , 200
        , 1700
        );

INSERT INTO departments VALUES 
        ( 20
        , 'Marketing'
        , 201
        , 1800
        );
                                
INSERT INTO departments VALUES 
        ( 30
        , 'Purchasing'
        , 114
        , 1700
	);
                
INSERT INTO departments VALUES 
        ( 40
        , 'Human Resources'
        , 203
        , 2400
        );

INSERT INTO departments VALUES 
        ( 50
        , 'Shipping'
        , 121
        , 1500
        );
                
INSERT INTO departments VALUES 
        ( 60 
        , 'IT'
        , 103
        , 1400
        );
                
INSERT INTO departments VALUES 
        ( 70 
        , 'Public Relations'
        , 204
        , 2700
        );
                
INSERT INTO departments VALUES 
        ( 80 
        , 'Sales'
        , 145
        , 2500
        );
                
INSERT INTO departments VALUES 
        ( 90 
        , 'Executive'
        , 100
        , 1700
        );

INSERT INTO departments VALUES 
        ( 100 
        , 'Finance'
        , 108
        , 1700
        );
                
INSERT INTO departments VALUES 
        ( 110 
        , 'Accounting'
        , 205
        , 1700
        );

INSERT INTO departments VALUES 
        ( 120 
        , 'Treasury'
        , NULL
        , 1700
        );

INSERT INTO departments VALUES 
        ( 130 
        , 'Corporate Tax'
        , NULL
        , 1700
        );

INSERT INTO departments VALUES 
        ( 140 
        , 'Control And Credit'
        , NULL
        , 1700
        );

INSERT INTO departments VALUES 
        ( 150 
        , 'Shareholder Services'
        , NULL
        , 1700
        );

INSERT INTO departments VALUES 
        ( 160 
        , 'Benefits'
        , NULL
        , 1700
        );

INSERT INTO departments VALUES 
        ( 170 
        , 'Manufacturing'
        , NULL
        , 1700
        );

INSERT INTO departments VALUES 
        ( 180 
        , 'Construction'
        , NULL
        , 1700
        );

INSERT INTO departments VALUES 
        ( 190 
        , 'Contracting'
        , NULL
        , 1700
        );

INSERT INTO departments VALUES 
        ( 200 
        , 'Operations'
        , NULL
        , 1700
        );

INSERT INTO departments VALUES 
        ( 210 
        , 'IT Support'
        , NULL
        , 1700
        );

INSERT INTO departments VALUES 
        ( 220 
        , 'NOC'
        , NULL
        , 1700
        );

INSERT INTO departments VALUES 
        ( 230 
        , 'IT Helpdesk'
        , NULL
        , 1700
        );

INSERT INTO departments VALUES 
        ( 240 
        , 'Government Sales'
        , NULL
        , 1700
        );

INSERT INTO departments VALUES 
        ( 250 
        , 'Retail Sales'
        , NULL
        , 1700
        );

INSERT INTO departments VALUES 
        ( 260 
        , 'Recruiting'
        , NULL
        , 1700
        );

INSERT INTO departments VALUES 
        ( 270 
        , 'Payroll'
        , NULL
        , 1700
        );


REM ***************************insert data into the JOBS table

Prompt ******  Populating JOBS table ....

INSERT INTO jobs VALUES 
        ( 'AD_PRES'
        , 'President'
        , 20000
        , 40000
        );
INSERT INTO jobs VALUES 
        ( 'AD_VP'
        , 'Administration Vice President'
        , 15000
        , 30000
        );

INSERT INTO jobs VALUES 
        ( 'AD_ASST'
        , 'Administration Assistant'
        , 3000
        , 6000
        );

INSERT INTO jobs VALUES 
        ( 'FI_MGR'
        , 'Finance Manager'
        , 8200
        , 16000
        );

INSERT INTO jobs VALUES 
        ( 'FI_ACCOUNT'
        , 'Accountant'
        , 4200
        , 9000
        );

INSERT INTO jobs VALUES 
        ( 'AC_MGR'
        , 'Accounting Manager'
        , 8200
        , 16000
        );

INSERT INTO jobs VALUES 
        ( 'AC_ACCOUNT'
        , 'Public Accountant'
        , 4200
        , 9000
        );
INSERT INTO jobs VALUES 
        ( 'SA_MAN'
        , 'Sales Manager'
        , 10000
        , 20000
        );

INSERT INTO jobs VALUES 
        ( 'SA_REP'
        , 'Sales Representative'
        , 6000
        , 12000
        );

INSERT INTO jobs VALUES 
        ( 'PU_MAN'
        , 'Purchasing Manager'
        , 8000
        , 15000
        );

INSERT INTO jobs VALUES 
        ( 'PU_CLERK'
        , 'Purchasing Clerk'
        , 2500
        , 5500
        );

INSERT INTO jobs VALUES 
        ( 'ST_MAN'
        , 'Stock Manager'
        , 5500
        , 8500
        );
INSERT INTO jobs VALUES 
        ( 'ST_CLERK'
        , 'Stock Clerk'
        , 2000
        , 5000
        );

INSERT INTO jobs VALUES 
        ( 'SH_CLERK'
        , 'Shipping Clerk'
        , 2500
        , 5500
        );

INSERT INTO jobs VALUES 
        ( 'IT_PROG'
        , 'Programmer'
        , 4000
        , 10000
        );

INSERT INTO jobs VALUES 
        ( 'MK_MAN'
        , 'Marketing Manager'
        , 9000
        , 15000
        );

INSERT INTO jobs VALUES 
        ( 'MK_REP'
        , 'Marketing Representative'
        , 4000
        , 9000
        );

INSERT INTO jobs VALUES 
        ( 'HR_REP'
        , 'Human Resources Representative'
        , 4000
        , 9000
        );

INSERT INTO jobs VALUES 
        ( 'PR_REP'
        , 'Public Relations Representative'
        , 4500
        , 10500
        );


REM ***************************insert data into the EMPLOYEES table

Prompt ******  Populating EMPLOYEES table ....

INSERT INTO employees VALUES 
        ( 100
        , 'Steven'
        , 'King'
        , 'SKING'
        , '515.123.4567'
        , TO_DATE('17-JUN-1987', 'dd-MON-yyyy')
        , 'AD_PRES'
        , 24000
        , NULL
        , NULL
        , 90
        );

INSERT INTO employees VALUES 
        ( 101
        , 'Neena'
        , 'Kochhar'
        , 'NKOCHHAR'
        , '515.123.4568'
        , TO_DATE('21-SEP-1989', 'dd-MON-yyyy')
        , 'AD_VP'
        , 17000
        , NULL
        , 100
        , 90
        );

INSERT INTO employees VALUES 
        ( 102
        , 'Lex'
        , 'De Haan'
        , 'LDEHAAN'
        , '515.123.4569'
        , TO_DATE('13-JAN-1993', 'dd-MON-yyyy')
        , 'AD_VP'
        , 17000
        , NULL
        , 100
        , 90
        );

INSERT INTO employees VALUES 
        ( 103
        , 'Alexander'
        , 'Hunold'
        , 'AHUNOLD'
        , '590.423.4567'
        , TO_DATE('03-JAN-1990', 'dd-MON-yyyy')
        , 'IT_PROG'
        , 9000
        , NULL
        , 102
        , 60
        );

INSERT INTO employees VALUES 
        ( 104
        , 'Bruce'
        , 'Ernst'
        , 'BERNST'
        , '590.423.4568'
        , TO_DATE('21-MAY-1991', 'dd-MON-yyyy')
        , 'IT_PROG'
        , 6000
        , NULL
        , 103
        , 60
        );

INSERT INTO employees VALUES 
        ( 105
        , 'David'
        , 'Austin'
        , 'DAUSTIN'
        , '590.423.4569'
        , TO_DATE('25-JUN-1997', 'dd-MON-yyyy')
        , 'IT_PROG'
        , 4800
        , NULL
        , 103
        , 60
        );

INSERT INTO employees VALUES 
        ( 106
        , 'Valli'
        , 'Pataballa'
        , 'VPATABAL'
        , '590.423.4560'
        , TO_DATE('05-FEB-1998', 'dd-MON-yyyy')
        , 'IT_PROG'
        , 4800
        , NULL
        , 103
        , 60
        );

INSERT INTO employees VALUES 
        ( 107
        , 'Diana'
        , 'Lorentz'
        , 'DLORENTZ'
        , '590.423.5567'
        , TO_DATE('07-FEB-1999', 'dd-MON-yyyy')
        , 'IT_PROG'
        , 4200
        , NULL
        , 103
        , 60
        );

INSERT INTO employees VALUES 
        ( 108
        , 'Nancy'
        , 'Greenberg'
        , 'NGREENBE'
        , '515.124.4569'
        , TO_DATE('17-AUG-1994', 'dd-MON-yyyy')
        , 'FI_MGR'
        , 12000
        , NULL
        , 101
        , 100
        );

INSERT INTO employees VALUES 
        ( 109
        , 'Daniel'
        , 'Faviet'
        , 'DFAVIET'
        , '515.124.4169'
        , TO_DATE('16-AUG-1994', 'dd-MON-yyyy')
        , 'FI_ACCOUNT'
        , 9000
        , NULL
        , 108
        , 100
        );

INSERT INTO employees VALUES 
        ( 110
        , 'John'
        , 'Chen'
        , 'JCHEN'
        , '515.124.4269'
        , TO_DATE('28-SEP-1997', 'dd-MON-yyyy')
        , 'FI_ACCOUNT'
        , 8200
        , NULL
        , 108
        , 100
        );

INSERT INTO employees VALUES 
        ( 111
        , 'Ismael'
        , 'Sciarra'
        , 'ISCIARRA'
        , '515.124.4369'
        , TO_DATE('30-SEP-1997', 'dd-MON-yyyy')
        , 'FI_ACCOUNT'
        , 7700
        , NULL
        , 108
        , 100
        );

INSERT INTO employees VALUES 
        ( 112
        , 'Jose Manuel'
        , 'Urman'
        , 'JMURMAN'
        , '515.124.4469'
        , TO_DATE('07-MAR-1998', 'dd-MON-yyyy')
        , 'FI_ACCOUNT'
        , 7800
        , NULL
        , 108
        , 100
        );

INSERT INTO employees VALUES 
        ( 113
        , 'Luis'
        , 'Popp'
        , 'LPOPP'
        , '515.124.4567'
        , TO_DATE('07-DEC-1999', 'dd-MON-yyyy')
        , 'FI_ACCOUNT'
        , 6900
        , NULL
        , 108
        , 100
        );

INSERT INTO employees VALUES 
        ( 114
        , 'Den'
        , 'Raphaely'
        , 'DRAPHEAL'
        , '515.127.4561'
        , TO_DATE('07-DEC-1994', 'dd-MON-yyyy')
        , 'PU_MAN'
        , 11000
        , NULL
        , 100
        , 30
        );

INSERT INTO employees VALUES 
        ( 115
        , 'Alexander'
        , 'Khoo'
        , 'AKHOO'
        , '515.127.4562'
        , TO_DATE('18-MAY-1995', 'dd-MON-yyyy')
        , 'PU_CLERK'
        , 3100
        , NULL
        , 114
        , 30
        );

INSERT INTO employees VALUES 
        ( 116
        , 'Shelli'
        , 'Baida'
        , 'SBAIDA'
        , '515.127.4563'
        , TO_DATE('24-DEC-1997', 'dd-MON-yyyy')
        , 'PU_CLERK'
        , 2900
        , NULL
        , 114
        , 30
        );

INSERT INTO employees VALUES 
        ( 117
        , 'Sigal'
        , 'Tobias'
        , 'STOBIAS'
        , '515.127.4564'
        , TO_DATE('24-JUL-1997', 'dd-MON-yyyy')
        , 'PU_CLERK'
        , 2800
        , NULL
        , 114
        , 30
        );

INSERT INTO employees VALUES 
        ( 118
        , 'Guy'
        , 'Himuro'
        , 'GHIMURO'
        , '515.127.4565'
        , TO_DATE('15-NOV-1998', 'dd-MON-yyyy')
        , 'PU_CLERK'
        , 2600
        , NULL
        , 114
        , 30
        );

INSERT INTO employees VALUES 
        ( 119
        , 'Karen'
        , 'Colmenares'
        , 'KCOLMENA'
        , '515.127.4566'
        , TO_DATE('10-AUG-1999', 'dd-MON-yyyy')
        , 'PU_CLERK'
        , 2500
        , NULL
        , 114
        , 30
        );

INSERT INTO employees VALUES 
        ( 120
        , 'Matthew'
        , 'Weiss'
        , 'MWEISS'
        , '650.123.1234'
        , TO_DATE('18-JUL-1996', 'dd-MON-yyyy')
        , 'ST_MAN'
        , 8000
        , NULL
        , 100
        , 50
        );

INSERT INTO employees VALUES 
        ( 121
        , 'Adam'
        , 'Fripp'
        , 'AFRIPP'
        , '650.123.2234'
        , TO_DATE('10-APR-1997', 'dd-MON-yyyy')
        , 'ST_MAN'
        , 8200
        , NULL
        , 100
        , 50
        );

INSERT INTO employees VALUES 
        ( 122
        , 'Payam'
        , 'Kaufling'
        , 'PKAUFLIN'
        , '650.123.3234'
        , TO_DATE('01-MAY-1995', 'dd-MON-yyyy')
        , 'ST_MAN'
        , 7900
        , NULL
        , 100
        , 50
        );

INSERT INTO employees VALUES 
        ( 123
        , 'Shanta'
        , 'Vollman'
        , 'SVOLLMAN'
        , '650.123.4234'
        , TO_DATE('10-OCT-1997', 'dd-MON-yyyy')
        , 'ST_MAN'
        , 6500
        , NULL
        , 100
        , 50
        );

INSERT INTO employees VALUES 
        ( 124
        , 'Kevin'
        , 'Mourgos'
        , 'KMOURGOS'
        , '650.123.5234'
        , TO_DATE('16-NOV-1999', 'dd-MON-yyyy')
        , 'ST_MAN'
        , 5800
        , NULL
        , 100
        , 50
        );

INSERT INTO employees VALUES 
        ( 125
        , 'Julia'
        , 'Nayer'
        , 'JNAYER'
        , '650.124.1214'
        , TO_DATE('16-JUL-1997', 'dd-MON-yyyy')
        , 'ST_CLERK'
        , 3200
        , NULL
        , 120
        , 50
        );

INSERT INTO employees VALUES 
        ( 126
        , 'Irene'
        , 'Mikkilineni'
        , 'IMIKKILI'
        , '650.124.1224'
        , TO_DATE('28-SEP-1998', 'dd-MON-yyyy')
        , 'ST_CLERK'
        , 2700
        , NULL
        , 120
        , 50
        );

INSERT INTO employees VALUES 
        ( 127
        , 'James'
        , 'Landry'
        , 'JLANDRY'
        , '650.124.1334'
        , TO_DATE('14-JAN-1999', 'dd-MON-yyyy')
        , 'ST_CLERK'
        , 2400
        , NULL
        , 120
        , 50
        );

INSERT INTO employees VALUES 
        ( 128
        , 'Steven'
        , 'Markle'
        , 'SMARKLE'
        , '650.124.1434'
        , TO_DATE('08-MAR-2000', 'dd-MON-yyyy')
        , 'ST_CLERK'
        , 2200
        , NULL
        , 120
        , 50
        );

INSERT INTO employees VALUES 
        ( 129
        , 'Laura'
        , 'Bissot'
        , 'LBISSOT'
        , '650.124.5234'
        , TO_DATE('20-AUG-1997', 'dd-MON-yyyy')
        , 'ST_CLERK'
        , 3300
        , NULL
        , 121
        , 50
        );

INSERT INTO employees VALUES 
        ( 130
        , 'Mozhe'
        , 'Atkinson'
        , 'MATKINSO'
        , '650.124.6234'
        , TO_DATE('30-OCT-1997', 'dd-MON-yyyy')
        , 'ST_CLERK'
        , 2800
        , NULL
        , 121
        , 50
        );

INSERT INTO employees VALUES 
        ( 131
        , 'James'
        , 'Marlow'
        , 'JAMRLOW'
        , '650.124.7234'
        , TO_DATE('16-FEB-1997', 'dd-MON-yyyy')
        , 'ST_CLERK'
        , 2500
        , NULL
        , 121
        , 50
        );

INSERT INTO employees VALUES 
        ( 132
        , 'TJ'
        , 'Olson'
        , 'TJOLSON'
        , '650.124.8234'
        , TO_DATE('10-APR-1999', 'dd-MON-yyyy')
        , 'ST_CLERK'
        , 2100
        , NULL
        , 121
        , 50
        );

INSERT INTO employees VALUES 
        ( 133
        , 'Jason'
        , 'Mallin'
        , 'JMALLIN'
        , '650.127.1934'
        , TO_DATE('14-JUN-1996', 'dd-MON-yyyy')
        , 'ST_CLERK'
        , 3300
        , NULL
        , 122
        , 50
        );

INSERT INTO employees VALUES 
        ( 134
        , 'Michael'
        , 'Rogers'
        , 'MROGERS'
        , '650.127.1834'
        , TO_DATE('26-AUG-1998', 'dd-MON-yyyy')
        , 'ST_CLERK'
        , 2900
        , NULL
        , 122
        , 50
        );

INSERT INTO employees VALUES 
        ( 135
        , 'Ki'
        , 'Gee'
        , 'KGEE'
        , '650.127.1734'
        , TO_DATE('12-DEC-1999', 'dd-MON-yyyy')
        , 'ST_CLERK'
        , 2400
        , NULL
        , 122
        , 50
        );

INSERT INTO employees VALUES 
        ( 136
        , 'Hazel'
        , 'Philtanker'
        , 'HPHILTAN'
        , '650.127.1634'
        , TO_DATE('06-FEB-2000', 'dd-MON-yyyy')
        , 'ST_CLERK'
        , 2200
        , NULL
        , 122
        , 50
        );

INSERT INTO employees VALUES 
        ( 137
        , 'Renske'
        , 'Ladwig'
        , 'RLADWIG'
        , '650.121.1234'
        , TO_DATE('14-JUL-1995', 'dd-MON-yyyy')
        , 'ST_CLERK'
        , 3600
        , NULL
        , 123
        , 50
        );

INSERT INTO employees VALUES 
        ( 138
        , 'Stephen'
        , 'Stiles'
        , 'SSTILES'
        , '650.121.2034'
        , TO_DATE('26-OCT-1997', 'dd-MON-yyyy')
        , 'ST_CLERK'
        , 3200
        , NULL
        , 123
        , 50
        );

INSERT INTO employees VALUES 
        ( 139
        , 'John'
        , 'Seo'
        , 'JSEO'
        , '650.121.2019'
        , TO_DATE('12-FEB-1998', 'dd-MON-yyyy')
        , 'ST_CLERK'
        , 2700
        , NULL
        , 123
        , 50
        );

INSERT INTO employees VALUES 
        ( 140
        , 'Joshua'
        , 'Patel'
        , 'JPATEL'
        , '650.121.1834'
        , TO_DATE('06-APR-1998', 'dd-MON-yyyy')
        , 'ST_CLERK'
        , 2500
        , NULL
        , 123
        , 50
        );

INSERT INTO employees VALUES 
        ( 141
        , 'Trenna'
        , 'Rajs'
        , 'TRAJS'
        , '650.121.8009'
        , TO_DATE('17-OCT-1995', 'dd-MON-yyyy')
        , 'ST_CLERK'
        , 3500
        , NULL
        , 124
        , 50
        );

INSERT INTO employees VALUES 
        ( 142
        , 'Curtis'
        , 'Davies'
        , 'CDAVIES'
        , '650.121.2994'
        , TO_DATE('29-JAN-1997', 'dd-MON-yyyy')
        , 'ST_CLERK'
        , 3100
        , NULL
        , 124
        , 50
        );

INSERT INTO employees VALUES 
        ( 143
        , 'Randall'
        , 'Matos'
        , 'RMATOS'
        , '650.121.2874'
        , TO_DATE('15-MAR-1998', 'dd-MON-yyyy')
        , 'ST_CLERK'
        , 2600
        , NULL
        , 124
        , 50
        );

INSERT INTO employees VALUES 
        ( 144
        , 'Peter'
        , 'Vargas'
        , 'PVARGAS'
        , '650.121.2004'
        , TO_DATE('09-JUL-1998', 'dd-MON-yyyy')
        , 'ST_CLERK'
        , 2500
        , NULL
        , 124
        , 50
        );

INSERT INTO employees VALUES 
        ( 145
        , 'John'
        , 'Russell'
        , 'JRUSSEL'
        , '011.44.1344.429268'
        , TO_DATE('01-OCT-1996', 'dd-MON-yyyy')
        , 'SA_MAN'
        , 14000
        , .4
        , 100
        , 80
        );

INSERT INTO employees VALUES 
        ( 146
        , 'Karen'
        , 'Partners'
        , 'KPARTNER'
        , '011.44.1344.467268'
        , TO_DATE('05-JAN-1997', 'dd-MON-yyyy')
        , 'SA_MAN'
        , 13500
        , .3
        , 100
        , 80
        );

INSERT INTO employees VALUES 
        ( 147
        , 'Alberto'
        , 'Errazuriz'
        , 'AERRAZUR'
        , '011.44.1344.429278'
        , TO_DATE('10-MAR-1997', 'dd-MON-yyyy')
        , 'SA_MAN'
        , 12000
        , .3
        , 100
        , 80
        );

INSERT INTO employees VALUES 
        ( 148
        , 'Gerald'
        , 'Cambrault'
        , 'GCAMBRAU'
        , '011.44.1344.619268'
        , TO_DATE('15-OCT-1999', 'dd-MON-yyyy')
        , 'SA_MAN'
        , 11000
        , .3
        , 100
        , 80
        );

INSERT INTO employees VALUES 
        ( 149
        , 'Eleni'
        , 'Zlotkey'
        , 'EZLOTKEY'
        , '011.44.1344.429018'
        , TO_DATE('29-JAN-2000', 'dd-MON-yyyy')
        , 'SA_MAN'
        , 10500
        , .2
        , 100
        , 80
        );

INSERT INTO employees VALUES 
        ( 150
        , 'Peter'
        , 'Tucker'
        , 'PTUCKER'
        , '011.44.1344.129268'
        , TO_DATE('30-JAN-1997', 'dd-MON-yyyy')
        , 'SA_REP'
        , 10000
        , .3
        , 145
        , 80
        );

INSERT INTO employees VALUES 
        ( 151
        , 'David'
        , 'Bernstein'
        , 'DBERNSTE'
        , '011.44.1344.345268'
        , TO_DATE('24-MAR-1997', 'dd-MON-yyyy')
        , 'SA_REP'
        , 9500
        , .25
        , 145
        , 80
        );

INSERT INTO employees VALUES 
        ( 152
        , 'Peter'
        , 'Hall'
        , 'PHALL'
        , '011.44.1344.478968'
        , TO_DATE('20-AUG-1997', 'dd-MON-yyyy')
        , 'SA_REP'
        , 9000
        , .25
        , 145
        , 80
        );

INSERT INTO employees VALUES 
        ( 153
        , 'Christopher'
        , 'Olsen'
        , 'COLSEN'
        , '011.44.1344.498718'
        , TO_DATE('30-MAR-1998', 'dd-MON-yyyy')
        , 'SA_REP'
        , 8000
        , .2
        , 145
        , 80
        );

INSERT INTO employees VALUES 
        ( 154
        , 'Nanette'
        , 'Cambrault'
        , 'NCAMBRAU'
        , '011.44.1344.987668'
        , TO_DATE('09-DEC-1998', 'dd-MON-yyyy')
        , 'SA_REP'
        , 7500
        , .2
        , 145
        , 80
        );

INSERT INTO employees VALUES 
        ( 155
        , 'Oliver'
        , 'Tuvault'
        , 'OTUVAULT'
        , '011.44.1344.486508'
        , TO_DATE('23-NOV-1999', 'dd-MON-yyyy')
        , 'SA_REP'
        , 7000
        , .15
        , 145
        , 80
        );

INSERT INTO employees VALUES 
        ( 156
        , 'Janette'
        , 'King'
        , 'JKING'
        , '011.44.1345.429268'
        , TO_DATE('30-JAN-1996', 'dd-MON-yyyy')
        , 'SA_REP'
        , 10000
        , .35
        , 146
        , 80
        );

INSERT INTO employees VALUES 
        ( 157
        , 'Patrick'
        , 'Sully'
        , 'PSULLY'
        , '011.44.1345.929268'
        , TO_DATE('04-MAR-1996', 'dd-MON-yyyy')
        , 'SA_REP'
        , 9500
        , .35
        , 146
        , 80
        );

INSERT INTO employees VALUES 
        ( 158
        , 'Allan'
        , 'McEwen'
        , 'AMCEWEN'
        , '011.44.1345.829268'
        , TO_DATE('01-AUG-1996', 'dd-MON-yyyy')
        , 'SA_REP'
        , 9000
        , .35
        , 146
        , 80
        );

INSERT INTO employees VALUES 
        ( 159
        , 'Lindsey'
        , 'Smith'
        , 'LSMITH'
        , '011.44.1345.729268'
        , TO_DATE('10-MAR-1997', 'dd-MON-yyyy')
        , 'SA_REP'
        , 8000
        , .3
        , 146
        , 80
        );

INSERT INTO employees VALUES 
        ( 160
        , 'Louise'
        , 'Doran'
        , 'LDORAN'
        , '011.44.1345.629268'
        , TO_DATE('15-DEC-1997', 'dd-MON-yyyy')
        , 'SA_REP'
        , 7500
        , .3
        , 146
        , 80
        );

INSERT INTO employees VALUES 
        ( 161
        , 'Sarath'
        , 'Sewall'
        , 'SSEWALL'
        , '011.44.1345.529268'
        , TO_DATE('03-NOV-1998', 'dd-MON-yyyy')
        , 'SA_REP'
        , 7000
        , .25
        , 146
        , 80
        );

INSERT INTO employees VALUES 
        ( 162
        , 'Clara'
        , 'Vishney'
        , 'CVISHNEY'
        , '011.44.1346.129268'
        , TO_DATE('11-NOV-1997', 'dd-MON-yyyy')
        , 'SA_REP'
        , 10500
        , .25
        , 147
        , 80
        );

INSERT INTO employees VALUES 
        ( 163
        , 'Danielle'
        , 'Greene'
        , 'DGREENE'
        , '011.44.1346.229268'
        , TO_DATE('19-MAR-1999', 'dd-MON-yyyy')
        , 'SA_REP'
        , 9500
        , .15
        , 147
        , 80
        );

INSERT INTO employees VALUES 
        ( 164
        , 'Mattea'
        , 'Marvins'
        , 'MMARVINS'
        , '011.44.1346.329268'
        , TO_DATE('24-JAN-2000', 'dd-MON-yyyy')
        , 'SA_REP'
        , 7200
        , .10
        , 147
        , 80
        );

INSERT INTO employees VALUES 
        ( 165
        , 'David'
        , 'Lee'
        , 'DLEE'
        , '011.44.1346.529268'
        , TO_DATE('23-FEB-2000', 'dd-MON-yyyy')
        , 'SA_REP'
        , 6800
        , .1
        , 147
        , 80
        );

INSERT INTO employees VALUES 
        ( 166
        , 'Sundar'
        , 'Ande'
        , 'SANDE'
        , '011.44.1346.629268'
        , TO_DATE('24-MAR-2000', 'dd-MON-yyyy')
        , 'SA_REP'
        , 6400
        , .10
        , 147
        , 80
        );

INSERT INTO employees VALUES 
        ( 167
        , 'Amit'
        , 'Banda'
        , 'ABANDA'
        , '011.44.1346.729268'
        , TO_DATE('21-APR-2000', 'dd-MON-yyyy')
        , 'SA_REP'
        , 6200
        , .10
        , 147
        , 80
        );

INSERT INTO employees VALUES 
        ( 168
        , 'Lisa'
        , 'Ozer'
        , 'LOZER'
        , '011.44.1343.929268'
        , TO_DATE('11-MAR-1997', 'dd-MON-yyyy')
        , 'SA_REP'
        , 11500
        , .25
        , 148
        , 80
        );

INSERT INTO employees VALUES 
        ( 169  
        , 'Harrison'
        , 'Bloom'
        , 'HBLOOM'
        , '011.44.1343.829268'
        , TO_DATE('23-MAR-1998', 'dd-MON-yyyy')
        , 'SA_REP'
        , 10000
        , .20
        , 148
        , 80
        );

INSERT INTO employees VALUES 
        ( 170
        , 'Tayler'
        , 'Fox'
        , 'TFOX'
        , '011.44.1343.729268'
        , TO_DATE('24-JAN-1998', 'dd-MON-yyyy')
        , 'SA_REP'
        , 9600
        , .20
        , 148
        , 80
        );

INSERT INTO employees VALUES 
        ( 171
        , 'William'
        , 'Smith'
        , 'WSMITH'
        , '011.44.1343.629268'
        , TO_DATE('23-FEB-1999', 'dd-MON-yyyy')
        , 'SA_REP'
        , 7400
        , .15
        , 148
        , 80
        );

INSERT INTO employees VALUES 
        ( 172
        , 'Elizabeth'
        , 'Bates'
        , 'EBATES'
        , '011.44.1343.529268'
        , TO_DATE('24-MAR-1999', 'dd-MON-yyyy')
        , 'SA_REP'
        , 7300
        , .15
        , 148
        , 80
        );

INSERT INTO employees VALUES 
        ( 173
        , 'Sundita'
        , 'Kumar'
        , 'SKUMAR'
        , '011.44.1343.329268'
        , TO_DATE('21-APR-2000', 'dd-MON-yyyy')
        , 'SA_REP'
        , 6100
        , .10
        , 148
        , 80
        );

INSERT INTO employees VALUES 
        ( 174
        , 'Ellen'
        , 'Abel'
        , 'EABEL'
        , '011.44.1644.429267'
        , TO_DATE('11-MAY-1996', 'dd-MON-yyyy')
        , 'SA_REP'
        , 11000
        , .30
        , 149
        , 80
        );

INSERT INTO employees VALUES 
        ( 175
        , 'Alyssa'
        , 'Hutton'
        , 'AHUTTON'
        , '011.44.1644.429266'
        , TO_DATE('19-MAR-1997', 'dd-MON-yyyy')
        , 'SA_REP'
        , 8800
        , .25
        , 149
        , 80
        );

INSERT INTO employees VALUES 
        ( 176
        , 'Jonathon'
        , 'Taylor'
        , 'JTAYLOR'
        , '011.44.1644.429265'
        , TO_DATE('24-MAR-1998', 'dd-MON-yyyy')
        , 'SA_REP'
        , 8600
        , .20
        , 149
        , 80
        );

INSERT INTO employees VALUES 
        ( 177
        , 'Jack'
        , 'Livingston'
        , 'JLIVINGS'
        , '011.44.1644.429264'
        , TO_DATE('23-APR-1998', 'dd-MON-yyyy')
        , 'SA_REP'
        , 8400
        , .20
        , 149
        , 80
        );

INSERT INTO employees VALUES 
        ( 178
        , 'Kimberely'
        , 'Grant'
        , 'KGRANT'
        , '011.44.1644.429263'
        , TO_DATE('24-MAY-1999', 'dd-MON-yyyy')
        , 'SA_REP'
        , 7000
        , .15
        , 149
        , NULL
        );

INSERT INTO employees VALUES 
        ( 179
        , 'Charles'
        , 'Johnson'
        , 'CJOHNSON'
        , '011.44.1644.429262'
        , TO_DATE('04-JAN-2000', 'dd-MON-yyyy')
        , 'SA_REP'
        , 6200
        , .10
        , 149
        , 80
        );

INSERT INTO employees VALUES 
        ( 180
        , 'Winston'
        , 'Taylor'
        , 'WTAYLOR'
        , '650.507.9876'
        , TO_DATE('24-JAN-1998', 'dd-MON-yyyy')
        , 'SH_CLERK'
        , 3200
        , NULL
        , 120
        , 50
        );

INSERT INTO employees VALUES 
        ( 181
        , 'Jean'
        , 'Fleaur'
        , 'JFLEAUR'
        , '650.507.9877'
        , TO_DATE('23-FEB-1998', 'dd-MON-yyyy')
        , 'SH_CLERK'
        , 3100
        , NULL
        , 120
        , 50
        );

INSERT INTO employees VALUES 
        ( 182
        , 'Martha'
        , 'Sullivan'
        , 'MSULLIVA'
        , '650.507.9878'
        , TO_DATE('21-JUN-1999', 'dd-MON-yyyy')
        , 'SH_CLERK'
        , 2500
        , NULL
        , 120
        , 50
        );

INSERT INTO employees VALUES 
        ( 183
        , 'Girard'
        , 'Geoni'
        , 'GGEONI'
        , '650.507.9879'
        , TO_DATE('03-FEB-2000', 'dd-MON-yyyy')
        , 'SH_CLERK'
        , 2800
        , NULL
        , 120
        , 50
        );

INSERT INTO employees VALUES 
        ( 184
        , 'Nandita'
        , 'Sarchand'
        , 'NSARCHAN'
        , '650.509.1876'
        , TO_DATE('27-JAN-1996', 'dd-MON-yyyy')
        , 'SH_CLERK'
        , 4200
        , NULL
        , 121
        , 50
        );

INSERT INTO employees VALUES 
        ( 185
        , 'Alexis'
        , 'Bull'
        , 'ABULL'
        , '650.509.2876'
        , TO_DATE('20-FEB-1997', 'dd-MON-yyyy')
        , 'SH_CLERK'
        , 4100
        , NULL
        , 121
        , 50
        );

INSERT INTO employees VALUES 
        ( 186
        , 'Julia'
        , 'Dellinger'
        , 'JDELLING'
        , '650.509.3876'
        , TO_DATE('24-JUN-1998', 'dd-MON-yyyy')
        , 'SH_CLERK'
        , 3400
        , NULL
        , 121
        , 50
        );

INSERT INTO employees VALUES 
        ( 187
        , 'Anthony'
        , 'Cabrio'
        , 'ACABRIO'
        , '650.509.4876'
        , TO_DATE('07-FEB-1999', 'dd-MON-yyyy')
        , 'SH_CLERK'
        , 3000
        , NULL
        , 121
        , 50
        );

INSERT INTO employees VALUES 
        ( 188
        , 'Kelly'
        , 'Chung'
        , 'KCHUNG'
        , '650.505.1876'
        , TO_DATE('14-JUN-1997', 'dd-MON-yyyy')
        , 'SH_CLERK'
        , 3800
        , NULL
        , 122
        , 50
        );

INSERT INTO employees VALUES 
        ( 189
        , 'Jennifer'
        , 'Dilly'
        , 'JDILLY'
        , '650.505.2876'
        , TO_DATE('13-AUG-1997', 'dd-MON-yyyy')
        , 'SH_CLERK'
        , 3600
        , NULL
        , 122
        , 50
        );

INSERT INTO employees VALUES 
        ( 190
        , 'Timothy'
        , 'Gates'
        , 'TGATES'
        , '650.505.3876'
        , TO_DATE('11-JUL-1998', 'dd-MON-yyyy')
        , 'SH_CLERK'
        , 2900
        , NULL
        , 122
        , 50
        );

INSERT INTO employees VALUES 
        ( 191
        , 'Randall'
        , 'Perkins'
        , 'RPERKINS'
        , '650.505.4876'
        , TO_DATE('19-DEC-1999', 'dd-MON-yyyy')
        , 'SH_CLERK'
        , 2500
        , NULL
        , 122
        , 50
        );

INSERT INTO employees VALUES 
        ( 192
        , 'Sarah'
        , 'Bell'
        , 'SBELL'
        , '650.501.1876'
        , TO_DATE('04-FEB-1996', 'dd-MON-yyyy')
        , 'SH_CLERK'
        , 4000
        , NULL
        , 123
        , 50
        );

INSERT INTO employees VALUES 
        ( 193
        , 'Britney'
        , 'Everett'
        , 'BEVERETT'
        , '650.501.2876'
        , TO_DATE('03-MAR-1997', 'dd-MON-yyyy')
        , 'SH_CLERK'
        , 3900
        , NULL
        , 123
        , 50
        );

INSERT INTO employees VALUES 
        ( 194
        , 'Samuel'
        , 'McCain'
        , 'SMCCAIN'
        , '650.501.3876'
        , TO_DATE('01-JUL-1998', 'dd-MON-yyyy')
        , 'SH_CLERK'
        , 3200
        , NULL
        , 123
        , 50
        );

INSERT INTO employees VALUES 
        ( 195
        , 'Vance'
        , 'Jones'
        , 'VJONES'
        , '650.501.4876'
        , TO_DATE('17-MAR-1999', 'dd-MON-yyyy')
        , 'SH_CLERK'
        , 2800
        , NULL
        , 123
        , 50
        );

INSERT INTO employees VALUES 
        ( 196
        , 'Alana'
        , 'Walsh'
        , 'AWALSH'
        , '650.507.9811'
        , TO_DATE('24-APR-1998', 'dd-MON-yyyy')
        , 'SH_CLERK'
        , 3100
        , NULL
        , 124
        , 50
        );

INSERT INTO employees VALUES 
        ( 197
        , 'Kevin'
        , 'Feeney'
        , 'KFEENEY'
        , '650.507.9822'
        , TO_DATE('23-MAY-1998', 'dd-MON-yyyy')
        , 'SH_CLERK'
        , 3000
        , NULL
        , 124
        , 50
        );

INSERT INTO employees VALUES 
        ( 198
        , 'Donald'
        , 'OConnell'
        , 'DOCONNEL'
        , '650.507.9833'
        , TO_DATE('21-JUN-1999', 'dd-MON-yyyy')
        , 'SH_CLERK'
        , 2600
        , NULL
        , 124
        , 50
        );

INSERT INTO employees VALUES 
        ( 199
        , 'Douglas'
        , 'Grant'
        , 'DGRANT'
        , '650.507.9844'
        , TO_DATE('13-JAN-2000', 'dd-MON-yyyy')
        , 'SH_CLERK'
        , 2600
        , NULL
        , 124
        , 50
        );

INSERT INTO employees VALUES 
        ( 200
        , 'Jennifer'
        , 'Whalen'
        , 'JWHALEN'
        , '515.123.4444'
        , TO_DATE('17-SEP-1987', 'dd-MON-yyyy')
        , 'AD_ASST'
        , 4400
        , NULL
        , 101
        , 10
        );

INSERT INTO employees VALUES 
        ( 201
        , 'Michael'
        , 'Hartstein'
        , 'MHARTSTE'
        , '515.123.5555'
        , TO_DATE('17-FEB-1996', 'dd-MON-yyyy')
        , 'MK_MAN'
        , 13000
        , NULL
        , 100
        , 20
        );

INSERT INTO employees VALUES 
        ( 202
        , 'Pat'
        , 'Fay'
        , 'PFAY'
        , '603.123.6666'
        , TO_DATE('17-AUG-1997', 'dd-MON-yyyy')
        , 'MK_REP'
        , 6000
        , NULL
        , 201
        , 20
        );

INSERT INTO employees VALUES 
        ( 203
        , 'Susan'
        , 'Mavris'
        , 'SMAVRIS'
        , '515.123.7777'
        , TO_DATE('07-JUN-1994', 'dd-MON-yyyy')
        , 'HR_REP'
        , 6500
        , NULL
        , 101
        , 40
        );

INSERT INTO employees VALUES 
        ( 204
        , 'Hermann'
        , 'Baer'
        , 'HBAER'
        , '515.123.8888'
        , TO_DATE('07-JUN-1994', 'dd-MON-yyyy')
        , 'PR_REP'
        , 10000
        , NULL
        , 101
        , 70
        );

INSERT INTO employees VALUES 
        ( 205
        , 'Shelley'
        , 'Higgins'
        , 'SHIGGINS'
        , '515.123.8080'
        , TO_DATE('07-JUN-1994', 'dd-MON-yyyy')
        , 'AC_MGR'
        , 12000
        , NULL
        , 101
        , 110
        );

INSERT INTO employees VALUES 
        ( 206
        , 'William'
        , 'Gietz'
        , 'WGIETZ'
        , '515.123.8181'
        , TO_DATE('07-JUN-1994', 'dd-MON-yyyy')
        , 'AC_ACCOUNT'
        , 8300
        , NULL
        , 205
        , 110
        );

REM ********* insert data into the JOB_HISTORY table

Prompt ******  Populating JOB_HISTORY table ....


INSERT INTO job_history
VALUES (102
       , TO_DATE('13-JAN-1993', 'dd-MON-yyyy')
       , TO_DATE('24-JUL-1998', 'dd-MON-yyyy')
       , 'IT_PROG'
       , 60);

INSERT INTO job_history
VALUES (101
       , TO_DATE('21-SEP-1989', 'dd-MON-yyyy')
       , TO_DATE('27-OCT-1993', 'dd-MON-yyyy')
       , 'AC_ACCOUNT'
       , 110);

INSERT INTO job_history
VALUES (101
       , TO_DATE('28-OCT-1993', 'dd-MON-yyyy')
       , TO_DATE('15-MAR-1997', 'dd-MON-yyyy')
       , 'AC_MGR'
       , 110);

INSERT INTO job_history
VALUES (201
       , TO_DATE('17-FEB-1996', 'dd-MON-yyyy')
       , TO_DATE('19-DEC-1999', 'dd-MON-yyyy')
       , 'MK_REP'
       , 20);

INSERT INTO job_history
VALUES  (114
        , TO_DATE('24-MAR-1998', 'dd-MON-yyyy')
        , TO_DATE('31-DEC-1999', 'dd-MON-yyyy')
        , 'ST_CLERK'
        , 50
        );

INSERT INTO job_history
VALUES  (122
        , TO_DATE('01-JAN-1999', 'dd-MON-yyyy')
        , TO_DATE('31-DEC-1999', 'dd-MON-yyyy')
        , 'ST_CLERK'
        , 50
        );

INSERT INTO job_history
VALUES  (200
        , TO_DATE('17-SEP-1987', 'dd-MON-yyyy')
        , TO_DATE('17-JUN-1993', 'dd-MON-yyyy')
        , 'AD_ASST'
        , 90
        );

INSERT INTO job_history
VALUES  (176
        , TO_DATE('24-MAR-1998', 'dd-MON-yyyy')
        , TO_DATE('31-DEC-1998', 'dd-MON-yyyy')
        , 'SA_REP'
        , 80
        );

INSERT INTO job_history
VALUES  (176
        , TO_DATE('01-JAN-1999', 'dd-MON-yyyy')
        , TO_DATE('31-DEC-1999', 'dd-MON-yyyy')
        , 'SA_MAN'
        , 80
        );

INSERT INTO job_history
VALUES  (200
        , TO_DATE('01-JUL-1994', 'dd-MON-yyyy')
        , TO_DATE('31-DEC-1998', 'dd-MON-yyyy')
        , 'AC_ACCOUNT'
        , 90
        );

REM enable integrity constraint to DEPARTMENTS

ALTER TABLE departments 
  ENABLE CONSTRAINT dept_mgr_fk;

COMMIT;


```

第二个表

```


SET FEEDBACK 1
SET NUMWIDTH 10
SET LINESIZE 80
SET TRIMSPOOL ON
SET TAB OFF
SET PAGESIZE 100
SET ECHO OFF 

REM ********************************************************************
REM Create the REGIONS table to hold region information for locations
REM HR.LOCATIONS table has a foreign key to this table.

Prompt ******  Creating REGIONS table ....

CREATE TABLE regions
    ( region_id      NUMBER 
       CONSTRAINT  region_id_nn NOT NULL 
    , region_name    VARCHAR2(25) 
    );

CREATE UNIQUE INDEX reg_id_pk
ON regions (region_id);

ALTER TABLE regions
ADD ( CONSTRAINT reg_id_pk
       		 PRIMARY KEY (region_id)
    ) ;

REM ********************************************************************
REM Create the COUNTRIES table to hold country information for customers
REM and company locations. 
REM OE.CUSTOMERS table and HR.LOCATIONS have a foreign key to this table.

Prompt ******  Creating COUNTRIES table ....

CREATE TABLE countries 
    ( country_id      CHAR(2) 
       CONSTRAINT  country_id_nn NOT NULL 
    , country_name    VARCHAR2(40) 
    , region_id       NUMBER 
    , CONSTRAINT     country_c_id_pk 
        	     PRIMARY KEY (country_id) 
    ) 
    ORGANIZATION INDEX; 

ALTER TABLE countries
ADD ( CONSTRAINT countr_reg_fk
        	 FOREIGN KEY (region_id)
          	  REFERENCES regions(region_id) 
    ) ;

REM ********************************************************************
REM Create the LOCATIONS table to hold address information for company departments.
REM HR.DEPARTMENTS has a foreign key to this table.

Prompt ******  Creating LOCATIONS table ....

CREATE TABLE locations
    ( location_id    NUMBER(4)
    , street_address VARCHAR2(40)
    , postal_code    VARCHAR2(12)
    , city       VARCHAR2(30)
	CONSTRAINT     loc_city_nn  NOT NULL
    , state_province VARCHAR2(25)
    , country_id     CHAR(2)
    ) ;

CREATE UNIQUE INDEX loc_id_pk
ON locations (location_id) ;

ALTER TABLE locations
ADD ( CONSTRAINT loc_id_pk
       		 PRIMARY KEY (location_id)
    , CONSTRAINT loc_c_id_fk
       		 FOREIGN KEY (country_id)
        	  REFERENCES countries(country_id) 
    ) ;

Rem 	Useful for any subsequent addition of rows to locations table
Rem 	Starts with 3300

CREATE SEQUENCE locations_seq
 START WITH     3300
 INCREMENT BY   100
 MAXVALUE       9900
 NOCACHE
 NOCYCLE;

REM ********************************************************************
REM Create the DEPARTMENTS table to hold company department information.
REM HR.EMPLOYEES and HR.JOB_HISTORY have a foreign key to this table.

Prompt ******  Creating DEPARTMENTS table ....

CREATE TABLE departments
    ( department_id    NUMBER(4)
    , department_name  VARCHAR2(30)
	CONSTRAINT  dept_name_nn  NOT NULL
    , manager_id       NUMBER(6)
    , location_id      NUMBER(4)
    ) ;

CREATE UNIQUE INDEX dept_id_pk
ON departments (department_id) ;

ALTER TABLE departments
ADD ( CONSTRAINT dept_id_pk
       		 PRIMARY KEY (department_id)
    , CONSTRAINT dept_loc_fk
       		 FOREIGN KEY (location_id)
        	  REFERENCES locations (location_id)
     ) ;

Rem 	Useful for any subsequent addition of rows to departments table
Rem 	Starts with 280 

CREATE SEQUENCE departments_seq
 START WITH     280
 INCREMENT BY   10
 MAXVALUE       9990
 NOCACHE
 NOCYCLE;

REM ********************************************************************
REM Create the JOBS table to hold the different names of job roles within the company.
REM HR.EMPLOYEES has a foreign key to this table.

Prompt ******  Creating JOBS table ....

CREATE TABLE jobs
    ( job_id         VARCHAR2(10)
    , job_title      VARCHAR2(35)
	CONSTRAINT     job_title_nn  NOT NULL
    , min_salary     NUMBER(6)
    , max_salary     NUMBER(6)
    ) ;

CREATE UNIQUE INDEX job_id_pk 
ON jobs (job_id) ;

ALTER TABLE jobs
ADD ( CONSTRAINT job_id_pk
      		 PRIMARY KEY(job_id)
    ) ;

REM ********************************************************************
REM Create the EMPLOYEES table to hold the employee personnel 
REM information for the company.
REM HR.EMPLOYEES has a self referencing foreign key to this table.

Prompt ******  Creating EMPLOYEES table ....

CREATE TABLE employees
    ( employee_id    NUMBER(6)
    , first_name     VARCHAR2(20)
    , last_name      VARCHAR2(25)
	 CONSTRAINT     emp_last_name_nn  NOT NULL
    , email          VARCHAR2(25)
	CONSTRAINT     emp_email_nn  NOT NULL
    , phone_number   VARCHAR2(20)
    , hire_date      DATE
	CONSTRAINT     emp_hire_date_nn  NOT NULL
    , job_id         VARCHAR2(10)
	CONSTRAINT     emp_job_nn  NOT NULL
    , salary         NUMBER(8,2)
    , commission_pct NUMBER(2,2)
    , manager_id     NUMBER(6)
    , department_id  NUMBER(4)
    , CONSTRAINT     emp_salary_min
                     CHECK (salary > 0) 
    , CONSTRAINT     emp_email_uk
                     UNIQUE (email)
    ) ;

CREATE UNIQUE INDEX emp_emp_id_pk
ON employees (employee_id) ;


ALTER TABLE employees
ADD ( CONSTRAINT     emp_emp_id_pk
                     PRIMARY KEY (employee_id)
    , CONSTRAINT     emp_dept_fk
                     FOREIGN KEY (department_id)
                      REFERENCES departments
    , CONSTRAINT     emp_job_fk
                     FOREIGN KEY (job_id)
                      REFERENCES jobs (job_id)
    , CONSTRAINT     emp_manager_fk
                     FOREIGN KEY (manager_id)
                      REFERENCES employees
    ) ;

ALTER TABLE departments
ADD ( CONSTRAINT dept_mgr_fk
      		 FOREIGN KEY (manager_id)
      		  REFERENCES employees (employee_id)
    ) ;


Rem 	Useful for any subsequent addition of rows to employees table
Rem 	Starts with 207 


CREATE SEQUENCE employees_seq
 START WITH     207
 INCREMENT BY   1
 NOCACHE
 NOCYCLE;

REM ********************************************************************
REM Create the JOB_HISTORY table to hold the history of jobs that 
REM employees have held in the past.
REM HR.JOBS, HR_DEPARTMENTS, and HR.EMPLOYEES have a foreign key to this table.

Prompt ******  Creating JOB_HISTORY table ....

CREATE TABLE job_history
    ( employee_id   NUMBER(6)
	 CONSTRAINT    jhist_employee_nn  NOT NULL
    , start_date    DATE
	CONSTRAINT    jhist_start_date_nn  NOT NULL
    , end_date      DATE
	CONSTRAINT    jhist_end_date_nn  NOT NULL
    , job_id        VARCHAR2(10)
	CONSTRAINT    jhist_job_nn  NOT NULL
    , department_id NUMBER(4)
    , CONSTRAINT    jhist_date_interval
                    CHECK (end_date > start_date)
    ) ;

CREATE UNIQUE INDEX jhist_emp_id_st_date_pk 
ON job_history (employee_id, start_date) ;

ALTER TABLE job_history
ADD ( CONSTRAINT jhist_emp_id_st_date_pk
      PRIMARY KEY (employee_id, start_date)
    , CONSTRAINT     jhist_job_fk
                     FOREIGN KEY (job_id)
                     REFERENCES jobs
    , CONSTRAINT     jhist_emp_fk
                     FOREIGN KEY (employee_id)
                     REFERENCES employees
    , CONSTRAINT     jhist_dept_fk
                     FOREIGN KEY (department_id)
                     REFERENCES departments
    ) ;

REM ********************************************************************
REM Create the EMP_DETAILS_VIEW that joins the employees, jobs, 
REM departments, jobs, countries, and locations table to provide details
REM about employees.

Prompt ******  Creating EMP_DETAILS_VIEW view ...

CREATE OR REPLACE VIEW emp_details_view
  (employee_id,
   job_id,
   manager_id,
   department_id,
   location_id,
   country_id,
   first_name,
   last_name,
   salary,
   commission_pct,
   department_name,
   job_title,
   city,
   state_province,
   country_name,
   region_name)
AS SELECT
  e.employee_id, 
  e.job_id, 
  e.manager_id, 
  e.department_id,
  d.location_id,
  l.country_id,
  e.first_name,
  e.last_name,
  e.salary,
  e.commission_pct,
  d.department_name,
  j.job_title,
  l.city,
  l.state_province,
  c.country_name,
  r.region_name
FROM
  employees e,
  departments d,
  jobs j,
  locations l,
  countries c,
  regions r
WHERE e.department_id = d.department_id
  AND d.location_id = l.location_id
  AND l.country_id = c.country_id
  AND c.region_id = r.region_id
  AND j.job_id = e.job_id 
WITH READ ONLY;

COMMIT;

```

第三个表

```


ALTER TABLE departments
DISABLE CONSTRAINT DEPT_MGR_FK;

ALTER TABLE job_history
DISABLE CONSTRAINT JHIST_EMP_FK;

DROP TRIGGER secure_employees;

DROP TRIGGER update_job_history;

DROP PROCEDURE add_job_history;

DROP PROCEDURE secure_dml;

DELETE FROM employees
WHERE manager_id IN (108, 114, 120, 121, 122, 123, 145, 146, 147, 148);

DELETE FROM employees
WHERE employee_id IN (114, 120, 121, 122, 123, 145, 146, 147, 148, 
                      196, 197, 198, 199, 105, 106, 108, 175, 177, 
                      179, 203, 204);

DELETE FROM locations
WHERE location_id NOT IN 
  (SELECT DISTINCT location_id
   FROM departments);

DELETE FROM countries
WHERE country_id NOT IN
  (SELECT country_id
   FROM locations);

DELETE FROM jobs
WHERE job_id NOT IN
  (SELECT job_id
   FROM employees);

DELETE FROM departments
WHERE department_id NOT IN 
  (SELECT DISTINCT department_id
   FROM employees
   WHERE department_id IS NOT NULL);

UPDATE departments
SET manager_id = 124
WHERE department_id = 50;

UPDATE departments
SET manager_id = 149
WHERE department_id = 80;

DELETE FROM locations
WHERE location_id IN (2700, 2400);

UPDATE locations
SET street_address = '460 Bloor St. W.', 
    postal_code = 'ON M5S 1X8'
WHERE location_id = 1800;

ALTER TABLE departments
ENABLE CONSTRAINT DEPT_MGR_FK;

CREATE TABLE job_grades
(grade_level VARCHAR2(3),
 lowest_sal  NUMBER,
 highest_sal NUMBER);

INSERT INTO job_grades
VALUES ('A', 1000, 2999);

INSERT INTO job_grades
VALUES ('B', 3000, 5999);

INSERT INTO job_grades
VALUES('C', 6000, 9999);

INSERT INTO job_grades
VALUES('D', 10000, 14999);

INSERT INTO job_grades
VALUES('E', 15000, 24999);

INSERT INTO job_grades
VALUES('F', 25000, 40000);

INSERT INTO departments VALUES 
        ( 190 
        , 'Contracting'
        , NULL
        , 1700
        );

COMMIT;


```
这里给出几个表直接的关系

![这里写图片描述](http://img.blog.csdn.net/20171229171530298?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbW1oMTk4OTExMTM=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


----------



hibernate的配置文件。
```
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD//EN"
        "http://www.hibernate.org/dtd/hibernate-configuration-3.0.dtd">
<hibernate-configuration>
    <session-factory>
        <!--  链接数据库的基本信息 -->
        <property name="hibernate.connection.username">system</property>
        <property name="hibernate.connection.password">xxxx</property>
        <property name="hibernate.connection.driver_class">oracle.jdbc.driver.OracleDriver</property>
        <property name="hibernate.connection.url">jdbc:oracle:thin:@10.0.63.42:1521:orcl</property>

        <!-- 配置数据库的方言 -->
        <property name="hibernate.dialect">org.hibernate.dialect.Oracle10gDialect</property>
        <!--<property name="dialect">org.hibernate.dialect.MySQL5Dialect</property>-->
        <!--<property name="dialect">org.hibernate.dialect.MySQLInnoDBDialect</property>-->

        <!--是否打印sql语句-->
        <property name="show_sql">true</property>

        <!--是否对sql语句格式化-->
        <property name="format_sql">true</property>

        <!--指定自动生成数据库表的策略，有4个值-->
        <property name="hbm2ddl.auto">update</property>

        <property name="connection.isolation">2</property>
        <!--<property name="use_identifier_rollback">true</property>-->

        <!--&lt;!&ndash;C3P0配置 &ndash;&gt;-->
        <!--<property name="hibernate.connection.provider_class">-->
            <!--org.hibernate.service.jdbc.connections.internal.C3P0ConnectionProvider-->
        <!--</property>-->
        <!--<property name="hibernate.c3p0.max_size">10</property>-->
        <!--<property name="hibernate.c3p0.min_size">5</property>-->
        <!--<property name="hibernate.c3p0.acquire_increment">5</property>-->
        <!--<property name="hibernate.c3p0.timeout">20000</property>-->
        <!--<property name="hibernate.c3p0.idle_test_period">20000</property>-->
        <!--<property name="hibernate.c3p0.max_statements">2</property>-->


        <property name="hibernate.jdbc.fetch_size">100</property>
        <property name="hibernate.jdbc.batch_size">30</property>


        <mapping resource="Department.hbm.xml"/>
        <mapping resource="Employee.hbm.xml"/>


    </session-factory>
</hibernate-configuration>
```

```
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">


<hibernate-mapping package="com.mamh.hibernate.hql.entities">

    <class name="Employee" table="hb_employee">
        <id name="id" type="java.lang.Integer">
            <column name="id"/>
            <generator class="native"/>
        </id>

        <property name="name" type="java.lang.String">
            <column name="name"/>
        </property>


        <property name="salary" type="float">
            <column name="salary"/>
        </property>

        <property name="email" type="java.lang.String">
            <column name="email"/>
        </property>

        <many-to-one name="dept" class="Department">
            <column name="dept_id"/>
        </many-to-one>


    </class>
</hibernate-mapping>
```

```
<?xml version='1.0' encoding='utf-8'?>
<!DOCTYPE hibernate-mapping PUBLIC
        "-//Hibernate/Hibernate Mapping DTD 3.0//EN"
        "http://www.hibernate.org/dtd/hibernate-mapping-3.0.dtd">


<hibernate-mapping package="com.mamh.hibernate.hql.entities">

    <class name="Department" table="hb_department">
        <id name="id"  type="java.lang.String">
            <column name="id" />
            <generator class="native"/>
        </id>
        <property name="name" type="java.lang.String">
            <column name="name"/>
        </property>

        <set name="emps" table="hb_employee" inverse="false" lazy="true">
            <key>
                <column name="dept_id"/>
            </key>
            <one-to-many class="Employee"/>
        </set>
    </class>
</hibernate-mapping>

```

```
package com.mamh.hibernate.hql.entities;


public class Department {
    private String id;
    private String name;

    private Set<Employee> emps = new HashSet<Employee>();

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Set<Employee> getEmps() {
        return emps;
    }

    public void setEmps(Set<Employee> emps) {
        this.emps = emps;
    }

    @Override
    public String toString() {
        return "Department{" +
                "id='" + id + '\'' +
                ", name='" + name + '\'' +
                '}';
    }
}

```

```
package com.mamh.hibernate.hql.entities;


public class Employee {
    private Integer id;
    private String name;
    private float salary;
    private String email;


    private Department dept;


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

    public float getSalary() {
        return salary;
    }

    public void setSalary(float salary) {
        this.salary = salary;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public Department getDept() {
        return dept;
    }

    public void setDept(Department dept) {
        this.dept = dept;
    }

    @Override
    public String toString() {
        return "Employee{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", salary=" + salary +
                ", email='" + email + '\'' +
                ", dept=" + dept +
                '}'+'\n';
    }
}

```

这2个类都是比较简单的。


----------

```
package com.mamh.hibernate.demo;


public class HibernateHqlTest {
    SessionFactory sessionFactory = null;
    private Session session = null;
    private Transaction transaction = null;


    @Before
    public void init() {
        System.out.println("=init=");
        //1.创建一个SessionFactory 对象，创建session的工厂的一个类
        //1.1创建一个Configuration对象，对应hibernate的基本配置信息和对象关系映射信息
        Configuration configuration = new Configuration().configure();
        //1.2创建一个ServiceRegistry对象，hibernate4.x新添加的对象，hibernate任何的配置和服务都要在该对象中注册才能有效。
        ServiceRegistry serviceRegistry = new ServiceRegistryBuilder().applySettings(configuration.getProperties()).buildServiceRegistry();
        //1.3
        sessionFactory = configuration.buildSessionFactory(serviceRegistry);
        //2.创建一个Session对象,这个和jdbc中的connection很类似
        session = sessionFactory.openSession();
        //3.开启事务
        transaction = session.beginTransaction();
    }



    @After
    public void destroy() {
        System.out.println("=destroy=");
        //5.提交事务
        transaction.commit();

        //6.关闭session对象
        session.close();

        //7.关闭SessionFactory 对象
        sessionFactory.close();

    }

}




```
添加测试hql查询类
```




    @Test
    public void testHQL(){

        String hql = "from Employee e where e.salary > ? and e.email like ?";
        Query query = session.createQuery(hql);

        query.setFloat(0, 6000).setString(1, "%A%");

        List list = query.list();
        System.out.println(list);


    }





```

```

Hibernate: 
    select
        employee0_.id as id1_,
        employee0_.name as name1_,
        employee0_.salary as salary1_,
        employee0_.email as email1_,
        employee0_.dept_id as dept5_1_ 
    from
        hb_employee employee0_ 
    where
        employee0_.salary>? 
        and (
            employee0_.email like ?
        )
Hibernate: 
    select
        department0_.id as id0_0_,
        department0_.name as name0_0_ 
    from
        hb_department department0_ 
    where
        department0_.id=?
Hibernate: 
    select
        department0_.id as id0_0_,
        department0_.name as name0_0_ 
    from
        hb_department department0_ 
    where
        department0_.id=?
Hibernate: 
    select
        department0_.id as id0_0_,
        department0_.name as name0_0_ 
    from
        hb_department department0_ 
    where
        department0_.id=?
Hibernate: 
    select
        department0_.id as id0_0_,
        department0_.name as name0_0_ 
    from
        hb_department department0_ 
    where
        department0_.id=?
Hibernate: 
    select
        department0_.id as id0_0_,
        department0_.name as name0_0_ 
    from
        hb_department department0_ 
    where
        department0_.id=?
Hibernate: 
    select
        department0_.id as id0_0_,
        department0_.name as name0_0_ 
    from
        hb_department department0_ 
    where
        department0_.id=?
Hibernate: 
    select
        department0_.id as id0_0_,
        department0_.name as name0_0_ 
    from
        hb_department department0_ 
    where
        department0_.id=?
Hibernate: 
    select
        department0_.id as id0_0_,
        department0_.name as name0_0_ 
    from
        hb_department department0_ 
    where
        department0_.id=?
Hibernate: 
    select
        department0_.id as id0_0_,
        department0_.name as name0_0_ 
    from
        hb_department department0_ 
    where
        department0_.id=?
[Employee{id=101, name='Kochhar', salary=17000.0, email='NKOCHHAR', dept=Department{id='90', name='Executive'}}
, Employee{id=102, name='De Haan', salary=17000.0, email='LDEHAAN', dept=Department{id='90', name='Executive'}}
, Employee{id=103, name='Hunold', salary=9000.0, email='AHUNOLD', dept=Department{id='60', name='IT'}}
, Employee{id=109, name='Faviet', salary=9000.0, email='DFAVIET', dept=Department{id='100', name='Finance'}}
, Employee{id=111, name='Sciarra', salary=7700.0, email='ISCIARRA', dept=Department{id='100', name='Finance'}}
, Employee{id=112, name='Urman', salary=7800.0, email='JMURMAN', dept=Department{id='100', name='Finance'}}
, Employee{id=114, name='Raphaely', salary=11000.0, email='DRAPHEAL', dept=Department{id='30', name='Purchasing'}}
, Employee{id=121, name='Fripp', salary=8200.0, email='AFRIPP', dept=Department{id='50', name='Shipping'}}
, Employee{id=122, name='Kaufling', salary=7900.0, email='PKAUFLIN', dept=Department{id='50', name='Shipping'}}
, Employee{id=123, name='Vollman', salary=6500.0, email='SVOLLMAN', dept=Department{id='50', name='Shipping'}}
, Employee{id=146, name='Partners', salary=13500.0, email='KPARTNER', dept=Department{id='80', name='Sales'}}
, Employee{id=147, name='Errazuriz', salary=12000.0, email='AERRAZUR', dept=Department{id='80', name='Sales'}}
, Employee{id=148, name='Cambrault', salary=11000.0, email='GCAMBRAU', dept=Department{id='80', name='Sales'}}
, Employee{id=152, name='Hall', salary=9000.0, email='PHALL', dept=Department{id='80', name='Sales'}}
, Employee{id=154, name='Cambrault', salary=7500.0, email='NCAMBRAU', dept=Department{id='80', name='Sales'}}
, Employee{id=155, name='Tuvault', salary=7000.0, email='OTUVAULT', dept=Department{id='80', name='Sales'}}
, Employee{id=158, name='McEwen', salary=9000.0, email='AMCEWEN', dept=Department{id='80', name='Sales'}}
, Employee{id=160, name='Doran', salary=7500.0, email='LDORAN', dept=Department{id='80', name='Sales'}}
, Employee{id=161, name='Sewall', salary=7000.0, email='SSEWALL', dept=Department{id='80', name='Sales'}}
, Employee{id=164, name='Marvins', salary=7200.0, email='MMARVINS', dept=Department{id='80', name='Sales'}}
, Employee{id=166, name='Ande', salary=6400.0, email='SANDE', dept=Department{id='80', name='Sales'}}
, Employee{id=167, name='Banda', salary=6200.0, email='ABANDA', dept=Department{id='80', name='Sales'}}
, Employee{id=172, name='Bates', salary=7300.0, email='EBATES', dept=Department{id='80', name='Sales'}}
, Employee{id=173, name='Kumar', salary=6100.0, email='SKUMAR', dept=Department{id='80', name='Sales'}}
, Employee{id=174, name='Abel', salary=11000.0, email='EABEL', dept=Department{id='80', name='Sales'}}
, Employee{id=175, name='Hutton', salary=8800.0, email='AHUTTON', dept=Department{id='80', name='Sales'}}
, Employee{id=176, name='Taylor', salary=8600.0, email='JTAYLOR', dept=Department{id='80', name='Sales'}}
, Employee{id=178, name='Grant', salary=7000.0, email='KGRANT', dept=null}
, Employee{id=201, name='Hartstein', salary=13000.0, email='MHARTSTE', dept=Department{id='20', name='Marketing'}}
, Employee{id=203, name='Mavris', salary=6500.0, email='SMAVRIS', dept=Department{id='40', name='Human Resources'}}
, Employee{id=204, name='Baer', salary=10000.0, email='HBAER', dept=Department{id='70', name='Public Relations'}}
]

=destroy=
```


参数使用命名参数的一种形式
```

    @Test
    public void testHQL1() {  //参数使用命名参数
        String hql = "from Employee e where e.salary > :sal and e.email like :email";
        Query query = session.createQuery(hql);


        query.setFloat("sal", 6000).setString("email", "%B%");

        List list = query.list();
        System.out.println(list);
        System.out.println(list.size());
    }
```

还可以使用order by
```

    @Test
    public void testHQL1() {  //参数使用命名参数
        String hql = "from Employee e where e.salary > :sal and e.email like :email  order by e.salary";
        Query query = session.createQuery(hql);


        query.setFloat("sal", 6000).setString("email", "%B%");

        List list = query.list();
        System.out.println(list);
        System.out.println(list.size());
    }

Hibernate: 

    select
        employee0_.id as id1_,
        employee0_.name as name1_,
        employee0_.salary as salary1_,
        employee0_.email as email1_,
        employee0_.dept_id as dept5_1_ 
    from
        hb_employee employee0_ 
    where
        employee0_.salary>? 
        and (
            employee0_.email like ?
        ) 
    order by
        employee0_.salary
```

使用实体类作为查询条件

```
    @Test
    public void testHQL1() {  //参数使用命名参数
        String hql = "from Employee e where e.salary > :sal and e.email like :email and e.dept = :dept order by e.salary";
        Query query = session.createQuery(hql);


        Department dept=new Department();
        dept.setId(80);
        query.setFloat("sal", 6000)
                .setString("email", "%B%")
                .setEntity("dept", dept);

        List list = query.list();
        //System.out.println(list);
        System.out.println(list.size());
    }
Hibernate: 
    select
        employee0_.id as id1_,
        employee0_.name as name1_,
        employee0_.salary as salary1_,
        employee0_.email as email1_,
        employee0_.dept_id as dept5_1_ 
    from
        hb_employee employee0_ 
    where
        employee0_.salary>? 
        and (
            employee0_.email like ?
        ) 
        and employee0_.dept_id=? 
    order by
        employee0_.salary
7
=destroy=
```


----------


分页查询

```

    @Test
    public void testHQL2(){//分页查询
        String hql = "from Employee";
        Query query = session.createQuery(hql);
        int pageNo = 3;
        int pageSize = 5;

        query.setFirstResult((pageNo - 1) * pageSize);
        query.setMaxResults(pageSize);
        List list = query.list();
        System.out.println(list);


    }




Hibernate: 
    select
        * 
    from
        ( select
            row_.*,
            rownum rownum_ 
        from
            ( select
                employee0_.id as id1_,
                employee0_.name as name1_,
                employee0_.salary as salary1_,
                employee0_.email as email1_,
                employee0_.dept_id as dept5_1_ 
            from
                hb_employee employee0_ ) row_ 
        where
            rownum <= ?
        ) 
    where
        rownum_ > ?
Hibernate: 
    select
        department0_.id as id0_0_,
        department0_.name as name0_0_ 
    from
        hb_department department0_ 
    where
        department0_.id=?
Hibernate: 
    select
        department0_.id as id0_0_,
        department0_.name as name0_0_ 
    from
        hb_department department0_ 
    where
        department0_.id=?
[Employee{id=110, name='Chen', salary=8200.0, email='JCHEN', dept=Department{id='100', name='Finance'}}
, Employee{id=111, name='Sciarra', salary=7700.0, email='ISCIARRA', dept=Department{id='100', name='Finance'}}
, Employee{id=112, name='Urman', salary=7800.0, email='JMURMAN', dept=Department{id='100', name='Finance'}}
, Employee{id=113, name='Popp', salary=6900.0, email='LPOPP', dept=Department{id='100', name='Finance'}}
, Employee{id=114, name='Raphaely', salary=11000.0, email='DRAPHEAL', dept=Department{id='30', name='Purchasing'}}
]
=destroy=

从第三页开始，每页显示5条记录。这里结果正好是从110id开始的。
```