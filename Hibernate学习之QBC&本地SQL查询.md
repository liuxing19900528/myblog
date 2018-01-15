hibernate 学习之QBC和本地SQL查询

QBC查询是通过使用hibernate提供的QUeryByCriteria API 来查询对象，
这种API封装了SQL语句的动态拼装，对查询提供了更加面向对象的功能接口。

本地SQL查询来完善HQL不能覆盖所有的查询特性



马哥私房菜博客地址：https://github.com/mageSFC/myblog


```java



    @Test
    public void testQBC() {
        //1.创建一个criteria 对象
        Criteria criteria = session.createCriteria(Employee.class);
        //2. 添加查询条件，在qbc中查询条件使用criterion来表示
        //criterion可以通过Restrictions类的静态方法得到
        criteria.add(Restrictions.eq("email", "SKUMAR"));
        criteria.add(Restrictions.gt("salary", 5000f));

        Employee employee = (Employee) criteria.uniqueResult();

        System.out.println(employee);


    }


    @Test
    public void testQBC2() {
        //1.创建一个criteria 对象
        Criteria criteria = session.createCriteria(Employee.class);

        //and  的使用  Conjunction 本身就是一个criterion对象，其中还可以添加criterion对象
        Conjunction conjunction = Restrictions.conjunction();
        conjunction.add(Restrictions.like("name", "a", MatchMode.ANYWHERE));

        Department dept = new Department();
        dept.setId(20);
        conjunction.add(Restrictions.eq("dept", dept));
        System.out.println(conjunction);


        Disjunction disjunction = Restrictions.disjunction();
        disjunction.add(Restrictions.ge("salary", 6000f));
        disjunction.add(Restrictions.isNull("email"));

        criteria.add(disjunction).add(conjunction);

        List list = criteria.list();
        System.out.println(list);


    }

    @Test
    public void testQBC3() {
        //1.创建一个criteria 对象
        Criteria criteria = session.createCriteria(Employee.class);

        //统计查询
        criteria.setProjection(Projections.max("salary"));

        Float o = (Float) criteria.uniqueResult();
        System.out.println(o);


    }


    @Test
    public void testQBC4() {
        //1.创建一个criteria 对象
        Criteria criteria = session.createCriteria(Employee.class);

        //排序
        criteria.addOrder(Order.asc("salary"));
        criteria.addOrder(Order.desc("email"));

        int pageSize = 5;
        int pageNo = 3;

        criteria.setFirstResult((pageNo - 1) * pageSize)
                .setMaxResults(pageSize);
        List list = criteria.list();
        System.out.println(list);

    }


    @Test
    public void testNativeSql() {
        String sql = "INSERT INTO HB_DEPARTMENT VALUES(?, ?)";
        Query sqlQuery = session.createSQLQuery(sql);

        sqlQuery.setInteger(0, 280).setString(1, "马哥私房菜").executeUpdate();

    }

    @Test
    public void testHQLUpdate(){
        String sql = "delete from Department  d where d.id = :id";
        Query query = session.createQuery(sql);
        query.setInteger("id", 280).executeUpdate();
    }

```