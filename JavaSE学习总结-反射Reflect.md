**马哥的淘宝店:https://shop592330910.taobao.com/**



反射(Reflection)是Java 程序开发语言的特征之一，它允许运行中的 Java 程序获取自身的信息，并且可以操作类或对象的内部属性。
>Oracle官方对反射的解释是

>Reflection enables Java code to discover information about the fields, methods and constructors of loaded classes, and to use reflected 
>fields, methods, and constructors to operate on their underlying counterparts, within security restrictions.
>members declared by a given class. It also allows programs to suppress default reflective access control.

反射就是动态的创建对象实例，之前创建对象实例都是在代码中写死的，比如 new Person()，创建一个人的对象实例，
但是我不想创建Person类对象了，我想换一个Student类对象实例，你就需要修改源码，然后重新编译。

如果用到反射的话，你只需要告诉程序上面什么时候去创建Person对象实例，什么时候去创建Student对象实例，
这个你可以配置在某个配置文件里（例如很多的框架都是配置在xml文件中的），这样你就不需要修改源代码，
重新编译了，你根据自己的配置可以随时的切换创建某个类的对象实例。

反射最重要的用途就是开发各种通用框架。

Java的反射机制的实现要借助于4个类：class，Constructor，Field，Method;


```
public class Person {
    public String name;
    private int age;

    public Person() { 马哥的淘宝店:https://shop592330910.taobao.com/
    }


    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }


    public void show() {
        System.out.println("我是一个人！马哥的淘宝店:https://shop592330910.taobao.com/");
    }

    public void display(String nation) {
        System.out.println("我的国籍是：" + nation);
    }
}
```

```

import org.junit.Test;

import java.lang.reflect.Field;

public class TestPerson {

    @Test
    public void test1() {
        Class<Person> clazz = Person.class;

        try {
            Person p = clazz.newInstance();

            //public的属性可以直接这样获取到
            Field f1 = clazz.getField("name");
            f1.set(p, "xxxxxxxxx马哥的淘宝店:https://shop592330910.taobao.com/");


            System.out.println(p);

            //私有的属性这样获取不到
            Field f2 = clazz.getField("age");
            f2.set(p, 20);
            System.out.println(p);


            //私有的属性这样这样可以调用到
            Field f2 = clazz.getDeclaredField("age");
            f2.setAccessible(true);
            f2.set(p, 20);
            System.out.println(p);
            
            //调用里面的方法，这个是public的方法
            Method m1 = clazz.getMethod("show");
            m1.invoke(p);

            //调用里面的方法，这个是public的方法,这个方法有参数
            Method m2 = clazz.getMethod("display", String.class);
            m2.invoke(p, "s");


        } catch (Exception e) {
            e.printStackTrace();
        }


    }

}

```

java.lang.Class 类，是反射的源头，


有了Class实例以后，可以进行的操作：
>>创建对应的运行时类的对象
>>获取对应的运行时类的完整结构（包括：属性，方法，构造器，内部类，父类，所在包，异常，注解。。。。）
>>可以调用对应的运行时类的结构（包括属性，方法，构造器）
>>反射的应用：动态代理

获取Class实例的4中方法
1.调用运行时类本身的.class 属性

```
Class<Person> clazz = Person.class;

Class<String> clazz1 = String.class;
```

2.通过运行时类的对象获取，getClass()方法

```
Person p = new Person();
Class<Person> clazz = p.getClass();
```
3.通过Class.forName() 方法来获取

```
Class<Person> clazz = Class.forName("com.test.reflect.Person");
```
4.通过类加载器

```
ClassLoader classLoader = this.getClass().getClassLoader();
Class<Person> clazz = classLoader.loadClass("com.test.reflect.Person");
```







