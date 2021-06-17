# 反射的一些常用方法：

1.1、获取Class的实例(三种)：

   - Class c = 类名.class
   - Class c = Class.forName("类的全限定类名");
   - Class c = 对象.getClass();

1.2、获取对象的类名

  - String className = c.getName();　　　　　　//获取全限定类名
  - String className = c.getSimpleName();　　　//获取简单类名

1.3、获取Field(四个方法)：

- Field field = c.getField("属性名");　　　　 //该方法只能通过属性名获取public的属性
- Field[] field = c.getFields();　　　　　　　//获取所有的public属性数组
- Field field = c.getDeclaredField("属性名"); //获取类的属性，包括protected/private

1.4、获取Field的信息

- String name = field.getName();　　//获取属性名
- Class<?> type = field.getType();　　//获取属性的类型
- Object value = field.get(obj);　　　　//获取obj对象的field属性的值
- field.set(obj,Object value);　　　　//给obj对象的field属性赋值value

1.5、设置可访问性

- setAccessible(true);　　//可以用在被访问修饰符修饰的地方，
      //默认为false只能对public修饰的操作，设置true可对private修饰的操作

### .得到类对象Class

```java
public class Car {
    static {
        System.out.println("静态块");
    }
}
```

..例1

```java
public static void main(String[] args) {  
//得到Class对象的方式
        //1、类名.Class
        Class class1 = Car.class;
        //2、对象.getClass（）
        Car c1 = new Car();
        Class class2 = c1.getClass();
        //3、Class.forName(‘类全路径’)
        Class class3 = null;
        try {
            class3 = Class.forName("day6_4.classTest.Car");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
}
```

..例2

```java
    public static void main(String[] args) {
        Class c = Car.class;
        try {
            //调用该类的无参构造方法，产生该类的对象
            Object obj = c.getDeclaredConstructor().newInstance();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

....例3

```java
    public static void main(String[] args) {
        try {
            Class.forName("day6_4.classTest.Car");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
```

..

### .获得类中定义的属性

```java
public class Car {
    public int price;
    public String type;
    private int age;
}
```

..

```java
    public static void main(String[] args) {
        Car c1 = new Car();
        Class c = Car.class;
        //得到该类中所有的公有属性集合
//        Field[] fs = c.getFields();

        //得到该类中定义的属性集合，包含私有属性
        Field[] fs = c.getDeclaredFields();
        for (Field f : fs) {
            System.out.println(f.getName());
        }
    }
```

### .获得类中定义的方法

```java
public class Car {
    public void move(){
        System.out.println("移动");
    }

    private void speak(){
        System.out.println("气轰");
    }
}
```

..例一

```java
    public static void main(String[] args) {
        Class c = Car.class;
        //得到该类中所有的公有方法，包括继承的公有方法
//        Method[] ms = c.getMethods();

        //得到该类中定义的所有方法，不包括继承的方法
        Method[] ms = c.getDeclaredMethods();
        for (Method m : ms) {
            System.out.println(m.getName());
        }
    }
```

..例二

```java
    public static void main(String[] args) {
        String str = "day6_4.classTest.Car";
        try {
            Class c = Class.forName(str);
            Method[] ms = c.getMethods();
            for(Method m : ms){
                System.out.println(m.getName());
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

# 反射在工厂模式中的应用

### .业务层

```java
public interface IUserService {
    public void addUser();
}
```

..

```java
public class UserServiceImpl implements IUserService {
//    private IUserDao dao = new UserDaoImpl();
    private IUserDao dao = (IUserDao) Factory.getObject("IUserDao");

    @Override
    public void addUser() {
        dao.addUser();

    }

    public static void main(String[] args) {
        IUserService u = (IUserService) Factory.getObject("IUserService");
        u.addUser();
    }
}
```

### .持久层

```java
public interface IUserDao {
    public void addUser();
}
```

..

```java
public class UserDaoImpl implements IUserDao {
    @Override
    public void addUser() {
        System.out.println("完成用户添加");
    }
}
```

### .工厂

```java
import java.io.FileReader;
import java.io.IOException;
import java.util.Properties;

/**
 * 工厂类
 */
public class Factory {
    /**创建属性对象*/
    private static Properties pro = new Properties();

    /***/
    static{
        try {
            //加载属性文件
            pro.load(new FileReader("src/com/project/dao.txt"));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    /**
     * 根据接口名称，得到实现类对象
     * @param interfaceName 接口名称
     * @return 实现类对象
     */
    public static Object getObject(String interfaceName){
        //根据键对象，得到值对象
        String value= pro.getProperty(interfaceName);
//        System.out.println(value);


        try {
            //加载类，得到类模板
            Class c = Class.forName(value);
            //产生该类的对象
            return c.getDeclaredConstructor().newInstance();
        } catch (Exception e) {
            e.printStackTrace();
        }

        return null;
    }

    public static void main(String[] args) {
       Object obj =  Factory.getObject("IUserDao");
        System.out.println(obj);
    }
}

```

### .配置文件

```java
IUserDao=UserDaoImpl
IUserService=UserServiceImpl
```

# 完成对数据库的增删改查**

（要写sql语句）

### .数据库

```
DROP TABLE IF EXISTS t_man;
CREATE TABLE t_man(
	pk_manId INT PRIMARY KEY AUTO_INCREMENT,
	m_name VARCHAR(20),
	m_sex ENUM('男','女'),
	m_birth DATE,
	m_money INT
);

INSERT INTO t_man(m_name, m_sex, m_birth, m_money) VALUES("张三","男","1998-02-05",5000),
			("张三丰","男","1998-03-05",6000),
			("张力得","男","1998-04-05",7000),
			("张开发","女","1998-05-05",8000),
			("赵四","女","1998-06-05",9000),
			("张动方","女","1998-07-05",55000);

SELECT*FROM t_man;

```



### .实体类

```java
public class ManBean {
    private int id;
    private String name;
    private String sex;
    private LocalDate birth;
    private int money;

    public ManBean() {
    }

    public ManBean(String name, String sex, LocalDate birth, int money) {
        //id是主键、自动增长列，故在这个构造方法中没有id这个形参
        this.name = name;
        this.sex = sex;
        this.birth = birth;
        this.money = money;
    }
    
//还差toString、get/set方法 
```

### .配置文件

```java
IManDao=day6_4.reflectTest2.dao.ManDaoImpl.java
IManService=project
```

### .BaseDao

```java
import java.lang.reflect.Field;
import java.sql.*;
import java.time.LocalDate;
import java.util.ArrayList;
import java.util.List;

public class BaseDao {
    //加载驱动
    static{
        try {
            Class.forName("com.mysql.jdbc.Driver");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }

    //建立连接
    protected Connection con;
    //sql语句执行对象
    protected PreparedStatement ps;
    //结果集
    protected ResultSet rs;


    /**
     * 建立连接
     */
    public void setConnection(){
        try {
            this.con = DriverManager.getConnection("jdbc:mysql://localhost:6789/dormitory?characterEncoding=utf-8",
                    "root", "lovo");
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
    }

    /**
     * 关闭连接
     */
    public void closeConnection(){
        try {
            if(rs!=null){ //避免空指针
                rs.close();
            }
            ps.close();
            con.close();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
    }


    /**
     * 对数据库进行增删改操作
     * @param sql 更新sql语句
     * @param valueArray 值列表
     */
    public void updateData(String sql, Object...valueArray){
        this.setConnection();
        try {
            ps = con.prepareStatement(sql);
            for(int i=0; i<valueArray.length; i++){
                ps.setObject(i+1, valueArray[i]);
            }
            ps.executeUpdate();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }finally {
            this.closeConnection();
        }
    }

    /*
     * 查询
     * @param beanClass 集合中存放元素的类模板
     * @param sql 查询sql语句
     * @param valueArray 值列表
     * @return 集合
     */
//    方法一：效率低
    /*public List selectData(Class beanClass, String sql, Object...valueArray){
        List list = new ArrayList();

        this.setConnection();
        try {
            ps = con.prepareStatement(sql);
            if(valueArray !=null && valueArray.length != 0) {
                for (int i = 0; i < valueArray.length; i++) {
                    ps.setObject(i + 1, valueArray[i]);
                }
            }
            //得到属性列表，得到的是所有的属性，在查询部分列时就会有问题，用try块来解决
            Field[] farray = beanClass.getDeclaredFields();
            rs= ps.executeQuery();
            while (rs.next()){
                //产生该实体类对象
               Object beanObj = beanClass.getDeclaredConstructor().newInstance();
                for (int i = 0; i < farray.length; i++) {
                    //得到属性名
                    String fileName = farray[i].getName();
                    //这个try块用于查询部分列的情况，查询部分列时会报异常，把它捕获且忽略，继续运行
                    try {
//                    System.out.println(fileName);
                        //从结果集中得到该属性对应的值
                        Object value = rs.getObject(fileName);
                        if (value instanceof java.sql.Date) {
                            value = LocalDate.parse(value.toString());
                        }
                        //根据属性名得到属性对象
                        Field f = beanClass.getDeclaredField(fileName);
// 此次解释了为什么sql语句中列的别名要与bean类中的属性名相同，
//  Object value = rs.getObject(fileName);，Field f = beanClass.getDeclaredField(fileName);
//  因为这两个语句中的形参都是fileName

                        //忽略访问修饰符的检查
                        f.setAccessible(true);
                        //将beanObj中当前属性赋值为value
                        f.set(beanObj, value);
                    }catch (Exception e){
                        continue;
                    }
                }
                list.add(beanObj);
            }
        } catch (Exception throwables) {
            throwables.printStackTrace();
        }finally {
            this.closeConnection();
        }
        return list;
    }*/

//    方法二:提高效率

    public List selectData(Class beanClass, String sql, Object...valueArray){
        List list = new ArrayList();
        this.setConnection();
        try {
            ps = con.prepareStatement(sql);
            if(valueArray !=null && valueArray.length != 0) {
                for (int i = 0; i < valueArray.length; i++) {
                    ps.setObject(i + 1, valueArray[i]);
            }
        }
        rs= ps.executeQuery();
        //得到结果集检查对象
        ResultSetMetaData rm = rs.getMetaData();
        //得到查询列的个数
        int num = rm.getColumnCount();
        while (rs.next()){
            //产生该实体类对象
            Object beanObj = beanClass.getDeclaredConstructor().newInstance();
            for(int i=1; i<=num  ;i++){
                //得到指定编号对应的列名
//                System.out.println(rm.getColumnLabel(i));
                String columnName = rm.getColumnLabel(i);
                //从结果集中得到指定列的值
                Object value = rs.getObject(columnName);
//                Object value = rs.getObject(rm.getColumnLabel(i));

//                System.out.println(value);
                if (value instanceof java.sql.Date) {
                    value = LocalDate.parse(value.toString());
                }
//                ---------------------------------------------
                //为级联属性赋值
                //判断列名是否有点“.”。如果有点，表示关联对象赋值
                if(columnName.indexOf(".") != -1){
                    String[] array = columnName.split("[.]");

                    //得到实体类的直接属性对象
                    Field f1 = beanClass.getDeclaredField(array[0]);
                    //去掉修饰符检查
                    f1.setAccessible(true);
                    Object fieldObj = f1.get(beanObj);
                    if(f1.get(beanObj) == null) {
                        //产生直接属性对象
                        fieldObj = f1.getType().getDeclaredConstructor().newInstance();
                        //将产生的属性对象，设置在实体对象中
                        f1.set(beanObj,fieldObj);
                    }
                    //得到直接属性对象的属性对象
                    Field f2 = f1.getType().getDeclaredField(array[1]);
                    f2.setAccessible(true);
                    //将直接属性对象的属性对象赋值
                    f2.set(fieldObj,value);
                }
//                --------------------------------------------
                //得到指定列对应的属性对象
                Field f = beanClass.getDeclaredField(columnName);
                f.setAccessible(true);
                f.set(beanObj,value);
            }
            list.add(beanObj);
        }
    } catch (Exception throwables) {
        throwables.printStackTrace();
    }finally {
        this.closeConnection();
    }
    return list;
}

    public static void main(String[] args) {
        BaseDao dao = new BaseDao();
        dao.setConnection();
//        System.out.println(dao.selectData());
    }
}
```

### .持久层

```java
public interface IManDao {
    /**
     * 添加
     * @param man
     */
    public void add(ManBean man);


    /**
     * 按编号删除
     * @param id
     */
    public void del(int id);


    /**
     * 按编号修改工资
     * @param id
     * @param money
     */
    public void update(int id, int money);


    /**
     * 查询所有人
     * @return
     */
    public List<ManBean> findAll();

    /**
     * 按姓名查询
     * @param name
     * @return
     */
    public List<ManBean> findByName(String name);

    /**
     * 按生日查询
     * @param startDate
     * @param endDate
     * @return
     */
    public List<ManBean> findByBirth(LocalDate startDate, LocalDate endDate);

    /**
     * 按id查询
     * @param id
     * @return
     */
    public List<ManBean> findById(int id);

}
```

..

```java
public class ManDaoImpl extends BaseDao implements IManDao{
    @Override
    public void add(ManBean man) {
        this.updateData("INSERT INTO t_man(m_name, m_sex, m_birth, m_money) VALUES(?,?,?,?)",
                man.getName(), man.getSex(), man.getBirth(), man.getMoney());
    }

    @Override
    public void del(int id) {

        this.updateData("DELETE FROM t_man WHERE pk_manId = ?",id);
    }

    @Override
    public void update(int id, int money) {

        this.updateData("update t_man set m_money = ? where pk_manId = ?",money,id);
    }

    @Override
    public List<ManBean> findAll() {
        //这里的别名必须和实体Bean中的属性名相同
        return this.selectData(ManBean.class,"select pk_manId id, m_name name, m_sex sex, m_birth birth, m_money money from t_man");
//        return this.selectData(ManBean.class,"select pk_manId id,  m_birth birth, m_money money from t_man");
    }

    @Override
    public List<ManBean> findByName(String name) {
        return this.selectData(ManBean.class,"select pk_manId id, m_name name, m_sex sex, m_birth birth, m_money money from t_man where m_name like ?", "%"+name+"%");
    }

    @Override
    public List<ManBean> findByBirth(LocalDate startDate, LocalDate endDate) {
        return this.selectData(ManBean.class,"select pk_manId id, m_name name, m_sex sex, m_birth birth, m_money money from t_man where m_birth> ? and m_birth < ?", startDate, endDate);
    }

    @Override
    public List<ManBean> findById(int id) {
        return this.selectData(ManBean.class,"select pk_manId id, m_name name, m_sex sex, m_birth birth, m_money money from t_man where pk_manId = ?", id);
    }

    public static void main(String[] args) {
        IManDao dao = new ManDaoImpl();
//        dao.add(new ManBean("赵大夫","男",LocalDate.parse("1999-06-14"),5252));
//        dao.del(6);
//        dao.update(5,58588);

        System.out.println(dao.findAll());
//        System.out.println(dao.findByName("张"));
//        System.out.println(dao.findById(3));
    }
}
```

# ReflectTest

### 测试Field类中的方法

```java
public class MyObj {
    private int age;
}
```

..

```java
    public static void main(String[] args) {
        Class c = MyObj.class;

        try {
            //根据属性名获取属性对象
            Field f = c.getDeclaredField("age");
            //得到属性名
            System.out.println(f.getName());

            //得到属性的类型
            Class fclass = f.getType();
            System.out.println(fclass);

//            //产生该类的的对象
            Object obj = c.getDeclaredConstructor().newInstance();
//            //去掉访问修饰符检查若对象的属性是私有的，允许我们访问它
            f.setAccessible(true);

            //得到对象的属性值
            Object value = f.get(obj);
            System.out.println(value);

//            //设置对象的属性值
            f.set(obj, 8);
            Object value1 = f.get(obj);
            System.out.println(value1);
        }catch (Exception e){
            e.printStackTrace();
        }
    }
```

### 测试Method类

```java
public class MyObj {
    public void run(int speed){
        System.out.println("现在的速度是"+speed);
    }

    public void run(){
        System.out.println("速度是9");
    }
}
```

..

```java
    public static void main(String[] args) {
        Class c = MyObj.class;
        try {
            //产生该类的的对象
            Object obj = c.getDeclaredConstructor().newInstance();
            //得到指定方法名的对象
            Method m = c.getMethod("run");
            //执行该方法
            m.invoke(obj);

//------------------------------------------------------------
            //产生该类的的对象
            Object obj1 = c.getDeclaredConstructor().newInstance();
            //得到带int类型参数的方法对象
            Method m1 = c.getMethod("run",int.class);
            //执行该方法，并传入实参
            m1.invoke(obj1, 48);
        }catch (Exception e){
            e.printStackTrace();
        }
    }
```

### 自定义方法并调用

```java
public class MyObj{
    private int money = 3000;
    private String name = "008";

    public void run(int speed){
        System.out.println("现在的速度是"+speed);
    }

    public void run(){
        System.out.println("速度是9");
    }

    public int getMoney() {
        return money;
    }

    public void setMoney(int money) {
        this.money = money;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

..

```java
public class Test {
    /**
     * 根据属性名，调用get方法得到指定对象的属性值
     * @param obj 对象
     * @param fieldName 属性名
     * @return 属性值
     */
    public static Object getMethodReturn(Object obj, String fieldName){
        //根据属性名，得到方法名
        String methodName = "get" + fieldName.substring(0,1).toUpperCase()
                +fieldName.substring(1);
        //得到方法对象
        try {
            Method m = obj.getClass().getMethod(methodName);
            //执行方法，得到方法的返回值
            Object value = m.invoke(obj);
            return value;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }


    /**
     * 调用set方法，将obj对象的指定属性，赋值为value
     * @param obj
     * @param fieldName
     * @param value
     */
    public static void setMethod(Object obj, String fieldName, Object value){
        //根据属性名，得到方法名
        String methodName = "set" + fieldName.substring(0,1).toUpperCase()
                +fieldName.substring(1);
        try {
            //得到属性对象
            Field f = obj.getClass().getDeclaredField(fieldName);
            //得到方法对象
            Method m = obj.getClass().getMethod(methodName, f.getType());
            //执行方法
            m.invoke(obj,value);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        Class c = MyObj.class;
        try {
//           产生该类的的对象
            Object obj = c.getDeclaredConstructor().newInstance();
            Object valueObj = getMethodReturn(obj,"money");
            System.out.println("valueObj = " + valueObj);

            Object obj2 = c.getDeclaredConstructor().newInstance();
            Object valueObj2 = getMethodReturn(obj2,"name");
            System.out.println("valueObj2 = " + valueObj2);
//-----------------------------------------------------------------
            Object obj3 = c.getDeclaredConstructor().newInstance();
            setMethod(obj3,"money",50);

            Object valueObj3 = getMethodReturn(obj3,"money");
            System.out.println("valueObj3 = " + valueObj3);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

..

```java

```

..

```java

```

# 打印出xml文件中的信息

### .类

```java
@MyAnnotation(name = "奔驰", array = {3,4,5})
public class Car {
    private String type;
    private int price;
    private String color;

    public Car(String type, int price, String color) {
        this.type = type;
        this.price = price;
        this.color = color;
    }

    public Car() {
    }

    public String getType() {
        return type;
    }

    public void setType(String type) {
        this.type = type;
    }

    public int getPrice() {
        return price;
    }

    public void setPrice(int price) {
        this.price = price;
    }

    public String getColor() {
        return color;
    }

    public void setColor(String color) {
        this.color = color;
    }

    @Override
    public String toString() {
        return "Car{" +
                "type='" + type + '\'' +
                ", price=" + price +
                ", color='" + color + '\'' +
                '}' + "\n";
    }
}
```

### .xml类型文件

```java
<?xml version="1.0" encoding="utf-8" ?> <!-- 申明 -->
<carList> <!--根元素-->
    <car Table="奥迪"> <!--属性-->
        <price>5000</price>
        <color>白色</color>
    </car>

    <car Table="宝马"> <!--属性-->
        <price>6000</price>
        <color>红色</color>
<!--        <sql>select*from t_car where c_money &gt; 10000 and c_money &lt;20000 </sql> -->
        <sql>
            <![CDATA[
            select*from t_car where c_money > 10000 and c_money < 20000
            ]]>
        </sql>

    </car>

</carList>
```

### .注解

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.TYPE) //该注解只能在类中用
@Retention(RetentionPolicy.RUNTIME) //该注解的值能在运行期间获取
public @interface MyAnnotation {
    public String name();

    public int age() default 20;

    public int[] array();

}
```

### .测试

```java
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;

public class CarTest {
    public static void main(String[] args) {
//        List<Car> carList = new ArrayList<>();
//
//        carList.add(new Car("奥迪", 5000, "白色"));
//        carList.add(new Car("宝马", 9000, "红色"));
//        System.out.println(carList);

//        ----------------------------------------
        Class carClass = Car.class;
        //判断该类中，是否有指定的注解
        if(carClass.isAnnotationPresent(MyAnnotation.class)){
            //得到指定的注解对象
            MyAnnotation m = (MyAnnotation) carClass.getAnnotation(MyAnnotation.class);
            //得到注解元素
            String name = m.name();
            int age = m.age();
            int[] array = m.array();
            System.out.println(name+ age+ Arrays.toString(array));
        }
    }
}
```

# 打印出xml文件中的信息

### .类

```java
public class Student {
    private String code;
    private String phone;
    private String name;
    private String edu;
    private int age;
    
    //toString  get/set
}
```

### .xml类型文件

```java
<?xml version="1.0" encoding="utf-8" ?>
<lovoClass>
    <student code="001" phone="15278945623">
        <name>张三</name>
        <edu>本科</edu>
        <age>20</age>
    </student>

    <student code="002">
        <name>李四</name>
        <edu>研究生</edu>
        <age>23</age>
    </student>

</lovoClass>
```

### .Test

```java
import java.util.ArrayList;
import java.util.List;

public class PressXml {
    public static void main(String[] args) {
        PressXml p = new PressXml();
        System.out.println(p.press());
    }

    public List<Student> press(){
        List<Student> list = new ArrayList<>();
        //产生SAX解析器对象
        SAXReader sax = new SAXReader();

        try {
            //读取指定XML文件，得到文档对象
            Document doc = sax.read("src/day6_7/XMLTest2/lovoClass.xml");
            //解析指定路径的节点，得到元素集合
            List<Element> elementList = doc.selectNodes("/lovoClass/student");

            for(Element ele : elementList){
                Student s = new Student();
                //将student标记中的code属性的值取出，赋值为对象的code属性
                s.setCode(ele.attributeValue("code"));
                s.setPhone(ele.attributeValue("phone"));

                //elementText表示得到子标记的内容
                s.setName(ele.elementText("name"));
                s.setEdu(ele.elementText("edu"));
                s.setAge(Integer.parseInt(ele.elementText("age")));

                list.add(s);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
        return list;
    }
}
```

# 注解完成数据库的增删改查**

### .实体类

```java
import java.time.LocalDate;
@Table("t_man")
public class ManBean {
    @Id
    @Column("pk_manId")
    private int id;
    @Column("m_name")
    private String name;
    @Column("m_sex")
    private String sex;
    @Column("m_birth")
    private LocalDate birth;
    @Column("m_money")
    private int money;

    public ManBean() {
    }

    public ManBean(String name, String sex, LocalDate birth, int money) {
        this.name = name;
        this.sex = sex;
        this.birth = birth;
        this.money = money;
    }
    //没写toString  get/set
}
```

### .注解

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Column {
    String value();
}
```

..

```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Id {
    //注解中没有元素，是标识性注解
}
```

..

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface Table {
    String value();
}
```

..

### .持久层

```java
import java.time.LocalDate;
import java.util.List;

public interface IManDao {
    /**
     * 添加
     * @param man
     */
    public void add(ManBean man);


    /**
     * 按编号删除
     * @param id
     */
    public void del(int id);


    /**
     * 按编号修改工资
     * @param id
     * @param money
     */
    public void update(int id, int money);


    /**
     * 查询所有人
     * @return
     */
    public List<ManBean> findAll();

    /**
     * 按姓名查询
     * @param name
     * @return
     */
    public List<ManBean> findByName(String name);

    /**
     * 按生日查询
     * @param startDate
     * @param endDate
     * @return
     */
    public List<ManBean> findByBirth(LocalDate startDate, LocalDate endDate);

    /**
     * 按id查询
     * @param id
     * @return
     */
    public List<ManBean> findById(int id);

}
```

..

```java
public class ManDaoImpl extends BaseDao implements IManDao {
    @Override
    public void add(ManBean man) {

        this.insert(man);
    }

    @Override
    public void del(int id) {

        this.updateData("DELETE FROM t_man WHERE pk_manId = ?",id);
    }

    @Override
    public void update(int id, int money) {

        this.updateData("update t_man set m_money = ? where pk_manId = ?",money,id);
    }

    @Override
    public List<ManBean> findAll() {

        //利用注解实现
        return this.find(ManBean.class,"select*from t_man");
    }

    @Override
    public List<ManBean> findByName(String name) {
        return this.find(ManBean.class,"select pk_manId id, m_name name, m_sex sex, m_birth birth, m_money money from t_man where m_name like ?", "%"+name+"%");
    }

    @Override
    public List<ManBean> findByBirth(LocalDate startDate, LocalDate endDate) {
        return this.find(ManBean.class,"select pk_manId id, m_name name, m_sex sex, m_birth birth, m_money money from t_man where m_birth> ? and m_birth < ?", startDate, endDate);
    }

    @Override
    public List<ManBean> findById(int id) {
        return this.find(ManBean.class,"select pk_manId id, m_name name, m_sex sex, m_birth birth, m_money money from t_man where pk_manId = ?", id);
    }

    public static void main(String[] args) {
        IManDao dao = new ManDaoImpl();
        dao.add(new ManBean("赵大夫","男",LocalDate.parse("1999-06-14"),5252));
//        dao.del(6);
//        dao.update(5,58588);

//        System.out.println(dao.findAll());
//        System.out.println(dao.findByName("张"));
//        System.out.println(dao.findById(3));
    }
}
```

..

### .BaseDao

```java
import java.lang.reflect.Field;
import java.sql.*;
import java.time.LocalDate;
import java.util.ArrayList;
import java.util.List;

public class BaseDao {

    //加载驱动
    static{
        try {
            Class.forName("com.mysql.jdbc.Driver");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }

    //建立连接
    protected Connection con;

    //sql语句执行对象
    protected PreparedStatement ps;

    //结果集
    protected ResultSet rs;


    /**
     * 建立连接
     */
    public void setConnection(){
        try {
            this.con = DriverManager.getConnection("jdbc:mysql://localhost:6789/mydb?characterEncoding=utf-8",
                    "root", "lovo");
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
    }

    /**
     * 关闭连接
     */
    public void closeConnection(){
        try {
            if(rs!=null){ //避免空指针
                rs.close();
            }
            ps.close();
            con.close();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
    }


    /**
     * 利用反射对数据库进行增删改操作
     * @param sql 更新sql语句
     * @param valueArray 值列表
     */
    public void updateData(String sql, Object...valueArray){
        this.setConnection();
        try {
            ps = con.prepareStatement(sql);
            for(int i=0; i<valueArray.length; i++){
                ps.setObject(i+1, valueArray[i]);
            }

            ps.executeUpdate();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }finally {
            this.closeConnection();
        }
    }


    /**
     *使用注解完成添加操作
     * @param beanObject
     */
    public void insert(Object beanObject){
        String sql = "insrt into ";
        String valueStr = " values(";

        //得到实体类的类模板
        Class beanClass = beanObject.getClass();

        List<Object> valueList = new ArrayList<>();

        //得到该类的Table注解
        Table table = (Table) beanClass.getAnnotation(Table.class);
        sql += table.value() + "(";

        //得到属性列表
        Field[] farray = beanClass.getDeclaredFields();

        for (Field f : farray) {
            //判断该属性是否有Column注解
            //id号是自动增长列，不应该加进去
            if (f.isAnnotationPresent(Column.class) && f.isAnnotationPresent(Id.class) == false) {
                //取出Column注解
                Column c = f.getAnnotation(Column.class);
                String columnName = c.value();
                sql += columnName + ",";

                valueStr += "?,";

                //去掉修饰符检查
                f.setAccessible(true);

                //得到beanObj指定属性的值
                Object valueObj = null;
                try {
                    valueObj = f.get(beanObject);//用try-catch包起来
                } catch (IllegalAccessException e) {
                    e.printStackTrace();
                }
                valueList.add(valueObj);
            }
        }

        //截掉最后一个逗号
        sql = sql.substring(0,sql.length()-1);
        valueStr = valueStr.substring(0,valueStr.length()-1);

        sql += ")";
        valueStr += ")";

        sql += valueStr;

        //JDBC操作
        this.setConnection();
        try {
            ps = con.prepareStatement(sql);

            for(int i=0; i<valueList.size(); i++){
                ps.setObject(i+1, valueList.get(i));
            }
            ps.executeUpdate();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }finally {
            this.closeConnection();
        }
    }


    /**
     * 查询操作
     * @param beanClass 实体类
     * @param sql sql语句
     * @param valueArray 值列表
     * @return
     */
    public List find(Class beanClass, String sql, Object...valueArray){

        List list = new ArrayList();
        this.setConnection();
        try {
            ps = con.prepareStatement(sql);
            if(valueArray !=null && valueArray.length != 0) {
                for (int i = 0; i < valueArray.length; i++) {
                    ps.setObject(i + 1, valueArray[i]);
            }
        }
        rs= ps.executeQuery();
        //得到结果集检查对象
        ResultSetMetaData rm = rs.getMetaData();
        //得到查询列的个数
        int num = rm.getColumnCount();
        while (rs.next()){
            //产生该实体类对象
            Object beanObj = beanClass.getDeclaredConstructor().newInstance();
            for(int i=1; i<=num  ;i++){
                //得到指定编号对应的列名
//                System.out.println(rm.getColumnLabel(i));
                String columnName = rm.getColumnLabel(i);
                //从结果集中得到指定列的值
                Object value = rs.getObject(columnName);
                if (value instanceof Date) {
                    value = LocalDate.parse(value.toString());
                }
//------------------------------------
                Field f = getField(beanClass,columnName);
                f.setAccessible(true);
                f.set(beanObj, value);
//   -----------------------------------
            }
            list.add(beanObj);
        }
    } catch (Exception throwables) {
        throwables.printStackTrace();
    }finally {
        this.closeConnection();
    }
    return list;
}

    private Field getField(Class beanClass, String columnName){
        //得到该类中的属性列表
        Field[] farray = beanClass.getDeclaredFields();

        for(Field f : farray){
            //判断该属性是否有Column注解
            if(f.isAnnotationPresent(Column.class)){
                //得到该属性的Column注解
                Column column = f.getAnnotation(Column.class);
                //得到该注解的value值
                String cname = column.value();

                if(cname.equals(columnName)){
                    return f;
                }
            }
        }
        return null;
    }

    public static void main(String[] args) {
        BaseDao dao = new BaseDao();
        dao.setConnection();
        System.out.println(dao.con);
        dao.insert(new BaseDao());
//        System.out.println(dao.selectData());

    }
```

..

```java

```



# 静态代理

#### 接口

```
public interface IEat {
    public void eat();

    public void drink();
}
```

#### 接口实现类

```
public class Eat implements IEat{
    @Override
    public void eat() {
        System.out.println("吃串串");
    }

    @Override
    public void drink() {
        System.out.println("喝酒");
    }
}
```

#### 代理对象

```java
/**
 * 代理对象
 */
public class Proxy implements IEat{
    private IEat ie;
    //定义一个接口对象，完成初始化
    public Proxy(IEat ie){
        this.ie = ie;
    }

    @Override
    public void eat() {
        System.out.println("订餐");
        //调用目标对象的目标方法
        ie.eat();
        System.out.println("结账，开发票");
    }

    @Override
    public void drink() {
        System.out.println("订餐");
        //调用目标对象的目标方法
        ie.drink();
        System.out.println("结账，开发票");
    }
}
```

#### .测试

```java
public class Test {
    public static void main(String[] args) {
        IEat ie = new Proxy(new Eat());
        ie.eat();
        ie.drink();
    }
}
```

.

# 动态代理**

## 1.初识代理模式

老板吃饭，代理来订餐和结账

#### .接口

```java
public interface IEat {
    public void eat();
    public void drink();
}
```

#### .接口实现类

```java
public class Eat implements IEat{
    @Override
    public void eat() {
        System.out.println("吃满汉全席");
    }

    @Override
    public void drink() {
        System.out.println("喝农夫山泉");
    }
}
```

#### .代理对象

和业务层接口实现类放在同一个包中

```java
/**
 * 代理对象
 */
public class ProxyUtil implements InvocationHandler {

    /**目标对象*/
    private Object target;

    /**
     * 构造方法完成目标对象的初始化
     * @param target
     */
    public ProxyUtil(Object target){
        this.target = target;
    }

    /**
     * 产生代理对象
     * @return 代理对象
     */
    public Object getProxy(){
        return Proxy.newProxyInstance(
                target.getClass().getClassLoader(),//得到目标对象的类加载器
                target.getClass().getInterfaces(),//得到目标对象实现的接口列表
                this  //InvocationHandler实现类
                         //无论调用目标对象的什么方法，都会执行InvocationHandler的invoke
        );
    }

    /**
     * 执行方法
     * @param proxy 产生的代理对象
     * @param method 执行的目标方法
     * @param args 执行目标方法的实参
     * @return 方法执行结果
     * @throws Throwable
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("订餐");
        //执行目标对象的目标方法
        Object value = method.invoke(target,args);
        System.out.println("结账，开发票"+"\n");
        return value;
    }
}

```

#### .测试

```java
public class Test {
    public static void main(String[] args) {
        ProxyUtil util = new ProxyUtil(new Eat());
        IEat ie = (IEat) util.getProxy();
        ie.eat();
        ie.drink();
    }
}
```

## 2.接口属性实例化（仿Spring）

利用代理模式完成，通过扫描持久层实现类所在的文件夹来完成业务接口实现类中持久层接口属性的实例化，完成用户的添加和删除日志业务。

利用代理模式来完成接口的实力化是为了降低耦合

#### .业务层

.完成用户添加和删除日志操作

```java
public interface IUserService {
    public void addUser();
}
```

.

```java
/**
 * 仿造Spring书写的此代理模式
 */
//这两个接口的实现类到这个包里面去找
@Scan("day6_8.ProxyTest3.dao.impl")
public class UserServiceImpl implements IUserService {

    //这两个接口需要实例化，用@Scan这个注解去找
    //为什么不直接在这里实例化，直接实例化耦合性太高
    @Autowaire
    private IUserDao userDao;
    @Autowaire
    private ILogDao logDao;
    
    @Override
    public void addUser() {
        //先完成用户的添加，在完成日志的删除
        userDao.add();
        logDao.del();
    }
}
```

..

```java

```



#### .持久层

.用户日志接口

```java
public interface ILogDao {
    public void del();
}
```

.

```java
public class LogDaoImpl implements ILogDao {
    @Override
    public void del() {
        System.out.println("删除日志");
    }
}
```

.用户接口

```java
public interface IUserDao {
    public void add();
}
```

..

```java
public class UserDaoImpl implements IUserDao {
    @Override
    public void add() {
        System.out.println("添加用户");
    }
}
```

#### .注解

```java
/**
 * 用于标识属性需要做实例化，即“注入”
 */
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Autowaire {
}
```

..

```java
/**
 * 扫描指定包中的注解
 */
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface Scan {
    String value();
}

```

.

#### .代理对象

和业务层接口实现类放在同一个包中

```java
//运行时有错误，注意是错误不是异常，代码跟老师写的一样，好像是有个类有问题
//错误名称：NoClassDefFoundError
import java.io.File;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.lang.reflect.Field;
import java.net.URL;
import java.net.URLDecoder;
import java.util.ArrayList;
import java.util.List;

public class ServiceProxy implements InvocationHandler {

    /**目标对象*/
    private Object target;

    /**
     * 构造方法完成目标对象的初始化
     * @param target
     */
    public ServiceProxy(Object target){
        this.target = target;
    }

    /**
     * 产生代理对象
     * @return 代理对象
     */
    public Object getProxy(){
        return Proxy.newProxyInstance(
                target.getClass().getClassLoader(),//得到目标对象的类加载器
                target.getClass().getInterfaces(),//得到目标对象实现的接口列表
                this  //InvocationHandler实现类
                //无论调用目标对象的什么方法，都会执行InvocationHandler的invoke
        );
    }


 /*   *//**
     * 为了完成对UserServiceImpl实现类的dao属性的实例化
     * 没有@Scan注解时这样写
     * @param proxy
     * @param method
     * @param args
     * @return
     * @throws Throwable
     *//*
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //得到目标对象的类模板
        Class serviceClass = target.getClass();

        //得到目标对象的属性列表
        Field[] farray = serviceClass.getDeclaredFields();
        for(Field f : farray){
            if(f.isAnnotationPresent(Autowaire.class)){
                //修改访问修饰符，私有的也可以访问
                f.setAccessible(true);
                if(f.getType()== IUserDao.class){
                    f.set(target, new UserDaoImpl());
                }
                else if(f.getType()== ILogDao.class){
                    f.set(target, new LogDaoImpl());
                }
            }
        }
        //执行目标对象的目标方法
        Object obj = method.invoke(target, args);

        return obj;
    }
*/


    /**
     * 为了完成对UserServiceImpl实现类的dao属性的实例化
     * 有@Scan注解时这样写,参照Spring原理
     * @param proxy
     * @param method
     * @param args
     * @return
     * @throws Throwable
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        //得到目标对象的类模板
        Class serviceClass = target.getClass();
        Scan scan = (Scan)serviceClass.getAnnotation(Scan.class);
        //得到要扫描的包名
        String packageName = scan.value();
        List<Class> classList = this.getClassList(packageName);

        //得到目标对象的属性列表
        Field[] farray = serviceClass.getDeclaredFields();
        for(Field f : farray){
            if(f.isAnnotationPresent(Autowaire.class)){
                //修改访问修饰符，私有的也可以访问
                f.setAccessible(true);

                for(Class c : classList){
                    //判断实现类类型是否匹配属性类型
                    if(f.getType().isAssignableFrom(c)){
                        //产生实现类对象
                        Object obj = c.getDeclaredConstructor().newInstance();
                        f.set(target, obj);
                    }
                }
            }
        }
        //执行目标对象的目标方法
        Object obj = method.invoke(target, args);

        return obj;
    }


    /**
     * 通过包名得到该包中所有类的集合
     * @param packageName
     * @return
     */
    private List<Class> getClassList(String packageName){

        try {
            List<Class> classList = new ArrayList<>();
            String packageDirName = packageName.replace('.','/');
            URL url = Thread.currentThread().getContextClassLoader().getResources(packageDirName).nextElement();
            String filePath = URLDecoder.decode(url.getFile(),"utf-8");
            File dirFile = new File(filePath);
            String[] classFiles = dirFile.list();
            for(String f : classFiles){
                if(f.endsWith(".class")){
                    String fileName = f.substring(0, f.lastIndexOf(".class"));
                    Class c = Class.forName(packageName + "." + fileName);
                    classList.add(c);
                }
            }
            return classList;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return null;
    }



}

```

#### .测试

```java
public class Test {
    public static void main(String[] args) {
        ServiceProxy p = new ServiceProxy(new UserServiceImpl());
        IUserService service = (IUserService) p.getProxy();
        service.addUser();
         }
}
```

## 3.插入异常日志

利用代理模式完成异常日志的插入

#### .业务接口

```java
/**
 * 完成异常日志的插入
 */
public interface IClassService {
    public void add();
    public void del();
}
```

.

```java
public class ClassServiceImpl implements IClassService{
    @Override
    public void add() {
        int x = 9/0;
    }

    @Override
    public void del() {
        int x = Integer.parseInt("abc");

    }
}
```

#### .代理对象

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public class ServiceProxy implements InvocationHandler {

    /**目标对象*/
    private Object target;

    /**
     * 构造方法完成目标对象的初始化
     * @param target
     */
    public ServiceProxy(Object target){
        this.target = target;
    }

    /**
     * 产生代理对象
     * @return 代理对象
     */
    public Object getProxy(){
        return Proxy.newProxyInstance(
                target.getClass().getClassLoader(),//得到目标对象的类加载器
                target.getClass().getInterfaces(),//得到目标对象实现的接口列表
                this  //InvocationHandler实现类
                //无论调用目标对象的什么方法，都会执行InvocationHandler的invoke
        );
    }
    
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        try {
            return method.invoke(target, args);
        }catch (Exception e){
           Class exClass =  e.getCause().getClass();
           //为调错提供依据，什么时候出现的什么异常
            System.out.println("执行 " + target.getClass().getName()+" 的 " + method.getName() + " 抛出 " + exClass.getName());
        }
        return null;
    }
}
```

#### .测试

```java
public class Test {
    public static void main(String[] args) {
        ServiceProxy p = new ServiceProxy(new UserServiceImpl());
        IUserService service = (IUserService) p.getProxy();
        service.addUser();
    }
}
```

## 4.当前方法耗费多少时间

#### .接口

```java
 // 利用代理模式记录当前方法耗费多少时间
public interface IManDao {
    public void test();
}
```

.

```java
public class ManDaoImpl  implements IManDao {
    //这里的方法是随便写的
    @Override
    public void test() {
        double a =4;
        for(int i = 0; i < 10; i++){
            a ++;
        }
        System.out.println(a);
    }
}
```

#### .代理对象

```java
import java.io.FileWriter;
import java.io.Writer;
import java.lang.reflect.Field;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.time.LocalDate;
import java.time.LocalTime;
import java.util.List;

public class DaoProxy implements InvocationHandler {

    /**目标对象*/
    private Object target;

    /**
     * 构造方法完成目标对象的初始化
     * @param target
     */
    public DaoProxy(Object target){
        this.target = target;
    }

    /**
     * 产生代理对象
     * @return 代理对象
     */
    public Object getProxy(){
        return Proxy.newProxyInstance(
                target.getClass().getClassLoader(),//得到目标对象的类加载器
                target.getClass().getInterfaces(),//得到目标对象实现的接口列表
                this  //InvocationHandler实现类
                //无论调用目标对象的什么方法，都会执行InvocationHandler的invoke
        );
    }

    /**
     * @param proxy
     * @param method
     * @param args
     * @return
     * @throws Throwable
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {

        long before = System.currentTimeMillis();
        //执行目标对象的目标方法
        Object obj = method.invoke(target, args);
        long after = System.currentTimeMillis();
        String str = LocalTime.now() + " 执行" + target.getClass().getName() +" 的" + method.getName() + " 方法，耗时 " + (after - before);
        
        Writer w = null;
        w = new FileWriter(LocalDate.now() + ".txt",true);
        try{
            w.write(str + "\n");
            System.out.println(str);
        }catch(Exception e){
            e.printStackTrace();
        }finally {
            w.close();
        }

        return obj;
    }
}
```

#### .测试

```java
public class DaoProxyTest {
    public static void main(String[] args) {
        DaoProxy p = new DaoProxy(new ManDaoImpl());
        IManDao service = (IManDao) p.getProxy();
        service.test();
    }
}
```

## 5.异常日志的记录


* 使用代理模式实现异常日志的记录。
* 实现步骤：
* 1、创建数据库日志表：t_log，列名：日志编号、日志内容、日志日期
* 2、创建一个用户类，提供run()、work()、sleep()方法，其中，run(）方法抛出数组下标越界异常， work()方法抛出空指针异常，sleep方法正确运行。
* 3、创建代理类，对用户类进行代理。在用户类方法抛出异常时，添加日志记录，日志日期默认为当天，日志内容书写为：XXX类在HH:mm:ss，执行XX方法时，抛出XX异常
* 4、编写测试类，创建用户类的代理对象，完成目标方法的调用。

#### .数据库

```
USE mydb;
DROP TABLE IF EXISTS t_log;

CREATE TABLE t_log(
	pk_id INT PRIMARY KEY AUTO_INCREMENT,
	l_content VARCHAR(200),
	l_date DATE
);

SELECT*FROM t_log;
```



#### .实体类

```java
import java.time.LocalDate;
@Table("t_log")
public class LogBean {
    @Id
//    @Column("pk_id")
    /**编号*/
    private int logId;

    @Column("l_content")
    /**内容*/
    private String content;

    @Column("l_date")
    /**时间*/
    private LocalDate date=LocalDate.now();

    public LogBean() {
    }

    public LogBean(String content) {
        this.content = content;

    }
    //没写toString  get/set
}
```

.

#### .业务层

```java
public interface IUserService {
    public void run();
    public void work();
    public void sleep();
}
```

.

```java
public class UserServiceImpl implements IUserService {
    private IUserDao userDao = new UserDaoImpl();
    private ILogDao logDao = new LogDaoImpl();

    @Override
    public void run() {
        userDao.run();
    }

    @Override
    public void work() {
        userDao.work();
    }

    @Override
    public void sleep() {
userDao.sleep();
    }
}
```

..

```java
public interface ILogService {
    public void add(LogBean log);
}
```

..

```java
public class LogServiceImpl implements ILogService {

    private IUserDao userDao = new UserDaoImpl();
    private ILogDao logDao = new LogDaoImpl();

    @Override
    public void add(LogBean log) {
        logDao.add(log);
//        userDao.sleep();
//        userDao.work();
//        userDao.run();
    }
}
```

.

#### .持久层

```java
public interface IUserDao {
    public void run();
    public void work();
    public void sleep();
}
```

.

```java
public class UserDaoImpl extends BaseDao implements IUserDao {
    @Override
    public void run() {
        int[] array= {1,2,3};
        System.out.println(array[4]);;
    }

    @Override
    public void work() {
        String s=null;
//        System.out.println(s.length());
        System.out.println(s.equals(""));
    }

    @Override
    public void sleep() {
        System.out.println("睡觉");
    }
}
```

..

```java
public interface ILogDao {
    public void add(LogBean log);
}
```

..

```java
public class LogDaoImpl extends BaseDao implements ILogDao {
    @Override
    public void add(LogBean log) {
        this.insert(log);
    }
}
```

#### .BaseDao

```java
import java.lang.reflect.Field;
import java.sql.*;
import java.util.ArrayList;
import java.util.List;

public class BaseDao {
    //加载驱动
    static{
        try {
            Class.forName("com.mysql.jdbc.Driver");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }

    //建立连接
    protected Connection con;

    //sql语句执行对象
    protected PreparedStatement ps;

    //结果集
    protected ResultSet rs;

    /**
     * 建立连接
     */
    public void setConnection(){
        try {
            this.con = DriverManager.getConnection("jdbc:mysql://localhost:6789/mydb?characterEncoding=utf-8",
                    "root", "lovo");
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
    }

    /**
     * 关闭连接
     */
    public void closeConnection(){
        try {
            if(rs!=null){ //避免空指针
                rs.close();
            }
            ps.close();
            con.close();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
    }


    /**
     *使用注解完成数据库插入操作
     * @param beanObject
     */
    public void insert(Object beanObject){
//        System.out.println(beanObject);
        String sql = "insert into ";
        String valueStr = " values(";

        //得到实体类的类模板
        Class beanClass = beanObject.getClass();

        List<Object> valueList = new ArrayList<>();

        //得到该类的Table注解
        Table table = (Table) beanClass.getAnnotation(Table.class);
        sql += table.value() + "(";

        //得到属性列表
        Field[] farray = beanClass.getDeclaredFields();

        for (Field f : farray) {
            //判断该属性是否有Column注解
            //id号是自动增长列，不应该加进去
            if (f.isAnnotationPresent(Column.class) && f.isAnnotationPresent(Id.class) == false) {
                //取出Column注解
                Column c = f.getAnnotation(Column.class);
                String columnName = c.value();
                sql += columnName + ",";

                valueStr += "?,";

                //去掉修饰符检查
                f.setAccessible(true);

                //得到beanObj指定属性的值
                Object valueObj = null;
                try {
                    valueObj = f.get(beanObject);//用try-catch包起来
                } catch (IllegalAccessException e) {
                    e.printStackTrace();
                }
                valueList.add(valueObj);
            }
        }

        //截掉最后一个逗号
        sql = sql.substring(0,sql.length()-1);
        valueStr = valueStr.substring(0,valueStr.length()-1);

        sql += ")";
        valueStr += ")";

        sql += valueStr;

        //JDBC操作
        this.setConnection();
        try {
            ps = con.prepareStatement(sql);

            for(int i=0; i<valueList.size(); i++){
                ps.setObject(i+1, valueList.get(i));
            }
            ps.executeUpdate();
        } catch (Exception throwables) {
            throwables.printStackTrace();
        }finally {
            this.closeConnection();
        }
    }

    public static void main(String[] args) {
        BaseDao dao =new BaseDao();
//        dao.setConnection();
//        System.out.println(dao.con);
        dao.insert(new LogBean("睡觉"));
    }
}
```

.

#### .注解

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Column {
    String value();
}
```

..

```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Id {
}
```

..

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface Table {
    String value();
}
```

.



#### .代理对象

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.time.LocalDateTime;
import java.time.format.DateTimeFormatter;

public class ServiceProxy implements InvocationHandler {
    /**目标对象*/
    private Object target;

    /**
     * 构造方法完成目标对象的初始化
     * @param target
     */
    public ServiceProxy(Object target){
        this.target = target;
    }

    /**
     * 产生代理对象
     * @return 代理对象
     */
    public Object getProxy(){
        return Proxy.newProxyInstance(
                target.getClass().getClassLoader(),//得到目标对象的类加载器
                target.getClass().getInterfaces(),//得到目标对象实现的接口列表
                this  //InvocationHandler实现类
                //无论调用目标对象的什么方法，都会执行InvocationHandler的invoke
        );
    }

    /**
     * 为了完成对UserServiceImpl实现类的dao属性的实例化
     * 
     * @param proxy
     * @param method
     * @param args
     * @return
     * @throws Throwable
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        try {
            return method.invoke(target, args);
        }catch (Exception e){
//            Class exClass =  e.getCause().getClass();
            Class exceptionClass = e.getCause().getClass();
            String exceptionName = exceptionClass.getName();
            LocalDateTime nowDate = LocalDateTime.now();
            String str = DateTimeFormatter.ofPattern("hh:mm:ss").format(nowDate);
            //为调错提供依据，什么时候出现的什么异常
//            System.out.println(target.getClass().getName()+" 类在 " +str+ " ,执行 "+ method.getName() + " 方法时抛出了 " + exceptionName + "异常");
            String content = target.getClass().getName()+" 类在 " +str+ " ,执行 "+ method.getName() + " 方法时抛出了 " + exceptionName + "异常";
            ILogService service = new LogServiceImpl();
            service.add(new LogBean(content));
        }
        return null;
    }
}


//        //日志内容书写为：执行XXX类的XXX方法，耗时XX毫秒，返回“XXX”。
//        String content = "执行 "+ target.getClass().getName()+" 类的 "+ method.getName() + " 方法，耗时 " + (endTime - startTime) + "ms";
//        System.out.println(content);
//        ILogService service = new LogServiceImpl();
//        service.add(new LogBean(content));
```

#### .测试

```java
public class Test {
    public static void main(String[] args) {
        ServiceProxy s =new ServiceProxy(new UserServiceImpl());
       IUserService service=(IUserService)s.getProxy();
       service.run();

//        service.work();
//        service.sleep();
    }
}
```



.

```java

```

.

.

```java

```

.

# 单例模式

## .立即加载单例模式（饱汉）

```java
public class Man {
    //立即加载单例模式，当类一加载，马上产生该类对象

    //单例模式首先要建一个私有的构造方法，
    //目的：防治外界通过new来无限的新建对象
    private Man() {
        System.out.println("产生Man");
    }

    private static Man m = new Man();

    //再建一个静态方法，必须是静态的，只有静态的才能通过类名点的方式访问
    //使得每一次访问得到的都是同一个对象
    public static Man getMan() {
        return m;
    }
}
```

.

```java
public class Test {
    public static void main(String[] args) {
        //测试单例模式，产生的都是是同一个对象
        Man m1 = Man.getMan();
        Man m2 = Man.getMan();

        System.out.println(m1 == m2);

//  --------------------------
//        测试单例模式的立即加载
        try {
            Class.forName("day6_9.danli.Man");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```

.

## .延迟加载单例模式

```java
public class Man {
        // 延迟加载的单例模式
    // 当类加载时并不实例化该对象，调用方法时，再实例化
    private Man(){
        System.out.println("产生Man");
    }

    private static class Inner{
        private static Man m = new Man();
    }

    public static Man getMan(){
        return Inner.m;
    }
}
```


.

```java
public class Test {
    public static void main(String[] args) {
//        延迟加载测试，饿汉模式
        try {
            Class.forName("day6_9.danli.Man");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
//        当类加载时并不实例化该对象，调用方法时，再实例化
        Man m3 = Man.getMan();
    }
}
```



# 装饰器模式

去商店买咖啡，给咖啡加糖，加奶的时候价格都会变化，若使用继承来实现，感觉很垃圾

### .主料接口（咖啡）

```java
public interface IProduct {
     // 商品价格
    public int price();
     // 商品描述
    public String info();
}
```

### .辅料接口

```java
public interface IIngredients extends IProduct{
}
```

### .辅料类型

糖

```java
public class Sugar implements IIngredients{
    private IProduct ip;

    //通过此构造方法完成主料的初始化
    public Sugar(IProduct ip){
        this.ip = ip;
    }

    @Override
    public int price() {
        return ip.price()+2;
//        return ip.getPrice()+2;
    }

    @Override
    public String info() {
        return ip.info() + " 加上糖 ";
    }
}
```

..牛奶

```java
public class Milk implements IIngredients{
    private IProduct ip;

    //通过此构造方法完成主料的初始化
    public Milk(IProduct ip){
        this.ip = ip;
    }

    @Override
    public int price() {
        return this.ip.price() + 3;
    }

    @Override
    public String info() {
        return this.ip.info() + " 加奶";
    }
}
```

..

### .咖啡种类

.爱尔兰咖啡

```java
public class IrishCoffee implements IProduct{
    @Override
    public int price() {
        return 12;
    }

    @Override
    public String info() {
        return "爱尔兰咖啡";
    }
}
```

.蓝山咖啡

```java
public class BlueCoffee implements IProduct{
    @Override
    public int price() {
        return 10;
    }

    @Override
    public String info() {
        return "蓝山咖啡";
    }
}
```

### .测试

```java
public class Test {
    public static void main(String[] args) {
        IProduct p = new IrishCoffee();
        p = new Sugar(p);
        p = new Sugar(p);
        p = new Milk(p);

        //最终实现随着需求的变化，价格动态变化
        System.out.println(p.info() + " 价格: " + p.price());
    }
}
```

..

# 适配器模式

想实现打电话和拍照这两样功能，采用适配器模式，有两种实现方式。

第一种是继承适配器，先继承一个类，在聚合一个类
第二种是对象适配器，通过继承多个接口实现

### .接口

```java
public interface ISpeak {
    public void speak();
}
```

.

```java
public class Tel implements ISpeak{
    @Override
    public void speak() {
        System.out.println("打电话");
    }
}
```

.

```java
public interface ITackPic {
    public void tackPicture();
}
```

.

```java
public class Camera implements ITackPic{
    @Override
    public void tackPicture() {
        System.out.println("照相");
    }
}
```

### .第一种实现方式

```java
public class Phone extends Tel{
    //第一种方式，继承适配器
    //先继承一个类，再聚合一个类，以完成适配器模式

    //聚合一个照相的属性，将类Camera作为属性传入
    private ITackPic tackPic = new Camera();

    public void tackPicture(){
        tackPic.tackPicture();
    }

    public static void main(String[] args) {
        Phone p = new Phone();
        p.tackPicture();
        p.speak();
    }
}
```

### .第二种实现方式

```java
public class Iphone implements ISpeak, ITackPic{
    //第二种方式，对象适配器,通过继承多个接口实现

    private ISpeak s = new Tel();
    private ITackPic t = new Camera();

    @Override
    public void speak() {
        s.speak();
    }

    @Override
    public void tackPicture() {
        t.tackPicture();
    }

    public static void main(String[] args) {
        Iphone p = new Iphone();
        p.speak();
        p.tackPicture();
    }
}
```



# 观察者模式

### .商品

```java
import java.util.Observable;
public class Product extends Observable {
    //价格
    private int price;

    public int getPrice(){
        return price;
    }

    public void setPrice(int price){
        this.price = price;
        //设置变化点
        this.setChanged();
        //通知观察者价格的变化
        this.notifyObservers(price);
    }
}
```

### .代理商

```java
import java.util.Observable;
import java.util.Observer;

public class AgentA implements Observer {
    //代理商售卖商品的价格
    public int price;

    public int getPrice() {
        return price;
    }

    @Override
    public void update(Observable o, Object arg) {
        //arg是商品价格变化后的新值
        int newPrice = (Integer)arg;
        this.price = (int)(newPrice*1.2);

    }
}
```

.

```java
import java.util.Observable;
import java.util.Observer;

public class AgentB implements Observer {
    //代理商B的售卖价格
    private int price;

    public int getPrice() {
        return price;
    }

    @Override
    public void update(Observable o, Object arg) {
        int newPrice = (Integer)arg;
        this.price = (int)(newPrice*1.3);
    }
}
```

### .测试

```java
public class Test {
    public static void main(String[] args) {
        Product p = new Product();
        AgentA a = new AgentA();
        AgentB b = new AgentB();
        //添加观察者
//        p.addObserver(new AgentA());
        p.addObserver(a);
        p.addObserver(b);

        //借此实现代理商的售卖价格岁商品的价格变化而变化
        p.setPrice(1200);
        System.out.println(a.getPrice());
        System.out.println(b.getPrice());
        System.out.println("------+++++---------");
        p.setPrice(1400);
        System.out.println(a.getPrice());
        System.out.println(b.getPrice());
    }
}
```



```java

```

.