# 一、控制反转(Inversion of Control,缩写为IOC)
* 1、用来降低计算机代码之间的耦合度
* 2、最常见的方式是依赖注入(Dependency Injection,简称DI)、依赖查询(Dependency Lookup)
* 3、把对象创建和对象之间的调用过程,交给Spring进行管理
# 二、IOC底层原理
* 1、xml解析、工厂模式、反射
* 2、IOC思想基于IOC容器完成,IOC容器底层就是对象工厂
* 3、Spring提供IOC容器的两个接口:
  * BeanFactory(IOC容器基本实现,Spring内部使用的接口)
  * ApplicationContext(BeanFactory接口的子接口,提供了更多功能,一般由开发人员使用),加载配置文件时就会在配置文件进行创建对象
  * ApplicationContext的两个实现类:ClassPathXmlApplicationContext的参数写xml的文件名，而FileSystemXmlApplicationContext的参数是写磁盘路径。
# 三、IOC操作Bean管理
* 1、Bean管理指的是两个操作：
````
Spring创建对象
Spring注入属性
````
* 2、Bean管理操作有两种方式：
````
基于xml配置文件方式实现
基于注解方式实现
````
# 四、xml操作Bean
* 1、基于xml方式创建对象
````
<bean id="User" class="com.jch.spring5.User"></bean>
在spring配置文件中，使用bean标签，标签里面添加对应属性，就可以实现对象创建。

在bean标签有很多属性，常用属性：
id属性：唯一标识
class属性：类全路径（包类路径）
创建对象的时候，默认也是执行无参数构造方法完成对象创建。
````
* 2、基于xml方式注入属性
````
DI：依赖注入，就是注入属性
IOC和DI有什么区别：DI是IOC中的具体实现，DI表示依赖注入或注入属性，注入属性要在创建对象的基础之上完成。
````
````
第一种注入方式：使用set方法进行注入
public class Book {
    private String bookName;
    private String bAuthor;
    // set方法注入
    public void setBookName(String bookName) {
        this.bookName = bookName;
    }
    public void setbAuthor(String bAuthor) {
        this.bAuthor = bAuthor;
    }
}

<!--set方法注入属性-->
<bean id="book" class="com.jch.spring5.Book">
    <!--使用property完成属性注入
        name：类里面属性名称
        value：向属性注入的值
    -->
    <property name="bookName" value="白鹿原"></property>
    <property name="bAuthor" value="陈忠实"></property>
</bean>
````
````
第二种注入方式：使用有参构造进行注入
public class Order {
    private String name;
    private String address;

    public Order(String name, String address) {
        this.name = name;
        this.address = address;
    }

    public void test() {
        System.out.println(name + "--" + address);
    }
}

<!--有参构造注入属性-->
<bean id="order" class="com.jch.spring5.Order">
    <constructor-arg name="name" value="电脑"></constructor-arg>
    <constructor-arg name="address" value="中国"></constructor-arg>
</bean>
````
````
第三种注入方式：p名称空间注入，可以简化基于xml配置方式
第一步: 在配置文件中添加p名称空间 xmlns:p="http://www.springframework.org/schema/p"
第二步: <bean id="book" class="com.company.spring5.Book" p:bookName="平凡的世界" p:bAuthor="路遥"></bean>
````
* 3、xml注入其他类型属性
````
<bean id="book" class="com.company.spring5.Book">
    <!--设置属性为空-->
    <property name="bookName">
        <null/>
    </property>
    <!-- 属性值包含特殊符号
        1. 把<>进行转义 &lt; &gt;
        2. 把带特殊符号内容写到CDATA
    -->
    <property name="bAuthor">
        <value><![CDATA[<<南京>>]]></value>
    </property>
</bean>
````
* 4、注入属性-外部bean
````
public class UserServiceImpl implements UserService{
    private UserDao userDao;

    public void setUserDao(UserDao userDao) {
        this.userDao = userDao;
    }

     @Override
    public void add() {
        System.out.println("add..............");
        userDao.update();
    }
}
````
````
public class UserDaoImpl implements UserDao {
    @Override
    public void update() {
        System.out.println("update...........");
    }
}
````
````
<!-- service和dao对象创建-->
<bean id="userService" class="com.jch.spring5.service.UserService">
  <!-- 注入userDao对象
    name属性: 类里面属性名
    ref属性: 创建UserDao对象bean标签的id值
    <property name="userDao" ref="userDaoImpl"></property>
  -->
</bean>
<bean id="userDaoImpl" class="com.jch.spring5.dao.UserDaoImpl"></bean>
````
* 5、注入属性-内部bean和级联赋值
````
（1）一对多关系：部门和员工。一个部门有多个员工。
（2）在实体类之间表示一对多关系。
````
````
//部门类
public class Dept {
    private String dname;

    public void setDname(String dname) {
        this.dname = dname;
    }
}
````
````
//员工类
public class Emp {
    private String ename;
    private String gender;
    //员工属于某一个部门，使用对象形式表示
    private Dept dept;

    public void setEname(String ename) {
        this.ename = ename;
    }

    public void setGender(String gender) {
        this.gender = gender;
    }

    public void setDept(Dept dept) {
        this.dept = dept;
    }

    public void test() {
        System.out.println(ename+"::"+gender+"::"+dept);
    }
}
````
````
第一种写法
<bean id="emp" class="com.jch.spring5.bean.Emp">
  <property name="ename" value="jch"></property>
  <property name="gender" value="male"></property>
  <!-- 级联赋值-->
  <property name="dept" value="dept"></property>
</bean>

<bean id="dept" class="com.jch.spring5.bean.Dept">
  <property name="dname" value="财务部"></property>
</bean>
````
````
第二种写法
前提是Emp的dept属性要加上get方法，不然dept.dname获取不到。
<bean id="emp" class="com.jch.spring5.bean.Emp">
  <property name="ename" value="jch"></property>
  <property name="gender" value="male"></property>
  <!-- 级联赋值-->
  <property name="dept" value="dept"></property>
  <property name="dept.dname" value="技术部"></property>
</bean>

<bean id="dept" class="com.jch.spring5.bean.Dept">
  <property name="dname" value=""></property>
</bean>
````
* 6、注入集合类型属性
````
(1) 注入数组类型属性
(2) 注入List集合类型属性
(3) 注入Map集合类型属性
````
````
public class Student {
    // 数组类型
    private String[] courses;
    //list集合类型
    private List<String> list;
    //map集合类型
    private Map<String, String> maps;

	//编写set方法
}
````
````
<bean id="stu" class="com.jch.spring5.collection.Student">
  <!-- 数组类型属性注入-->
  <property name="courses">
    <array>
      <value>java</value>
      <value>数据库</value>
    </array>
  </property>
  <!-- list类型属性注入-->
  <property name="list">
    <list>
      <value>张三</value>
      <value>李四</value>
    </list>
  </property> 
  <!-- map类型属性注入-->
  <property name="maps">
    <map>
      <entry key="JAVA" value="java"></entry>
      <entry key="PHP" value="php"></entry>
    </map>
  </property>  
</bean>
````
* 7、list集合中存放对象类型
````
public class Student {
    private List<Course> courseList;
    public void setCourseList(List<Course> courseList) {
        this.courseList = courseList;
    }
}
````
````
<bean id="stu" class="com.jch.spring5.collection.Student">
  <!-- list类型属性注入-->
  <property name="courseList">
    <list>
      <ref bean="course1"></ref>
      <ref bean="course2"></ref>
    </list>
  </property>
</bean>

<bean id="course1" class="com.company.spring5.collection.Course">
  <property name="cname" value="Spring5课程"></property>
</bean>
<bean id="cours2" class="com.company.spring5.collection.Course">
  <property name="cname" value="Mybatis课程"></property>
</bean>
````
* 8、把集合注入部分提取出来，让别人也能使用这个集合
````
在spring配置文件中引入名称空间util
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
    xmlns:util="http://www.springframework.org/schema/util"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
     http://www.springframework.org/schema/beans/spring-beans-4.1.xsd
     http://www.springframework.org/schema/util
     http://www.springframework.org/schema/util/spring-util-4.1.xsd">

</beans>
````
````
<!-- 提取list集合类型属性注入-->
<util:list id="bookList">
  <value>book1</value>
  <value>book2</value>
  <value>book3</value>
</util:list>

<!-- 提取list集合类型属性注入使用-->
<bean id="book" class="com.jch.spring5.collection.Book">
  <property name="list" ref="bookList"></property>
</bean>
````
* 9、FactoryBean工厂bean
````
普通bean：在配置文件中定义bean类型就是返回类型
工厂bean：在配置文件定义bean类型可以和返回类型不
````
````
第一步，创建类，这个类作为工厂bean，实现接口FactoryBean
public class MyBean implements FactoryBean {

  @Override
  public Course getObject() throws Exception {
    Course course = new Course();
    course.setCname("jch");
    return course;
  }

  @Override
  public Class<?> getObjectType() {
    return null;
  }

  @Override
  public boolean isSingleton() {
    return false;
  }
}

第二步，实现接口里面的方法，在实现的方法中定义返回的bean类型
<bean id="myBean" class="com.company.spring5.factoryBean.MyBean"></bean>

@Test
public void test() {
  // 1、加载spring配置文件
  ApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");
  // 2、获取配置创建的对象
  Course course = context.getBean("myBean", Course.class);
  System.out.println(course);
}
````
# 五、Bean的作用域
````
1、在Spring中，设置创建bean实例默认是单实例对象。
2、在spring配置文件bean标签里面有属性scope用于设置单实例还是多实例。
  scope属性值
  singleton，表示单实例对象。
  prototype，表示多实例对象。
3、singleton和prototype的区别
  设置scope值是singleton的时候，加载spring配置文件就会创建单实例对象
  设置scope值是prototype的时候，在调用getBean方法创建多实例对象。
````
# 六、Bean生命周期
````
1、通过构造器创建Bean实例(无参数构造)
2、为Bean的属性设置值(调用set方法)
3、调用Bean的初始化方法(需要进行配置)
4、bean可以使用(对象获取到了)
5、当容器关闭时,调用Bean的销毁方法(需要配置销毁方法)

public class Orders {
  private String oname;

  public Orders() {
    System.out.println("第一步 通过无参构造创建bean实例");
  }

  public void setOname(String oname) {
    System.out.println("第二步 调用set方法设置属性值");
  }

  public void initMethod() {
    System.out.println("第三步 执行初始化方法");
  }

  public void destroyMethod() {
    System.out.println("第五步 执行销毁方法");
  }
}
````
````
上后置处理器，有七个步骤：
1、通过构造器创建bean实例（无参构造）
2、为bean的属性设置值和对其他bean引用（调用set方法）
3、把bean实例传递bean后置处理器的方法(初始化之前执行的方法)
4、调用bean的初始化方法（配置初始化方法）
5、把bean实例传递bean后置处理器的方法(初始化之后执行的方法)
6、使用bean（对象获取到了）
7、当容器关闭，调用bean的销毁方法（需要配置销毁的方法）

创建类实现接口BeanPostProcessor，创建后置处理器
public class MyBeanPost implements BeanPostProcessor {
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("在初始化之前执行的方法");
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("在初始化之后执行的方法");
        return bean;
    }
}
<!-- 配置后置处理器-->
<bean id="myBeanPost" class="com.jch.spring5.bean.MyBeanPost"/bean><>
````
# 七、Bean的自动装配
````
自动装配：指 Spring 容器在不使用 和 标签的情况下，可以自动装配（autowire）相互协作的 Bean 之间的关联关系，将一个 Bean 注入其他 Bean 的 Property 中。
````
* 1、使用byName方式(Bean的id与类名必须一致)
````
//创建Cat实体类
public class Cat {
    public void shout(){
        System.out.println("喵呜~");
    }
}
````
````
//创建Dog实体类
public class Dog {
    public void shout(){
        System.out.println("汪汪汪~");
    }
}
````
````
//创建People实体类
public class People {
    private Cat cat;
    private  Dog dog;
    private String name;
    public Cat getCat() {
        return cat;
    }
    public void setCat(Cat cat) {
        this.cat = cat;
    }
    public Dog getDog() {
        return dog;
    }
    public void setDog(Dog dog) {
        this.dog = dog;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    @Override
    public String toString() {
        return "People{" +
                "cat=" + cat +
                ", dog=" + dog +
                ", name='" + name + '\'' +
                '}';
    }
}
````
````
 <?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--编写Cat实体类的Bean信息-->
	  <bean id="cat" class="com.kuang.pojo.Cat"></bean>
    <!--编写Dog实体类的Bean信息-->
    <bean id="dog" class="com.kuang.pojo.Dog"></bean>
    <!--编写People实体类的Bean信息-->
    <bean id="people" class="com.kuang.pojo.People" autowire="byName">
        <!--给name属性设置值-->
        <property name="name" value="jch"></property>

        <!-- 自动注入可以省略这两个-->
        <!--给dog属性设置值：通过引入dog的Bean信息-->
        <!-- <property name="dog" ref="dog"></property> -->
        <!--给cat属性设置值：通过引入cat的Bean信息-->
        <!-- <property name="cat" ref="cat"></property> -->
    </bean>
</beans>       
````
````
使用byName方式自动装配时，首先需要保证所有Bean的id唯一
要引用的Bean的id和当前设置自动装配的Bean的set方法的值要一致
````
* 2、使用byType方式
````
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!--ByType可以省略bean的id-->
    <bean class="com.kuang.pojo.Cat"></bean>
    <bean  class="com.kuang.pojo.Dog"></bean>
  <!--使用ByType方式：
            会在Spring的IOC容器中自动查找,
            和要引用的对象的属性类型相同的的Bean-->
    <bean id="people" class="com.kuang.pojo.People" autowire="byType">
        <property name="name" value="华农兄弟"></property>
    </bean>
</beans> 

````
# 八、引入外部属性文件
````
配置数据库信息
1、配置德鲁伊连接池，引入德鲁伊依赖jar包
2、引入外部属性文件配置数据库连接池
jdbc.properties文件

prop.driveClass=com.mysql.jdbc.Driver
prop.url=jdbc:mysql://localhostt:3306/mysqlName
prop.username=root
prop.password=root

3、把外部properties属性文件引入到Spring配置文件
引入名称空间
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation=
               "http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">


4、引入外部属性文件，用标签赋值
<!--加载外部的properties文件  -->
    <c:property-placeholder location="classpath:jdbc.properties"/>

<!-- 配置连接池-->
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="${prop.driver}"></property>
        <property name="jdbcUrl" value="${prop.url}"></property>
        <property name="user" value="${prop.username}"></property>
        <property name="password" value="${prop.password}"></property>
</bean>
````
# 九、注解操作Bean
* 1、基于注解方式实现对象创建
````
1、@Component
2、@Service
3、@Controller
4、@Repository
````
````
第一步,引入依赖
spring-aop-version.jar
第二步,开启组件扫描,哪个类上有注解就进行创建
<context:component-scan base-package="com.jch.spring5"></context:component-scan>
第三步，创建类，在类上面添加创建对象注解
注解中的value属性可以省略不写，不写默认值是类名首字母小写
@Component(value="")
public class StudentService {
  public void add() {
    System.out.println("service add...");
  }
}
````
````
use-default-filters：表示是否使用默认filter，false表示使用自己配置的filter
context:include-filter: 设置扫描哪些内容
context:exclude-filter: 表示设置哪些内容不进行扫描
````
* 2、注解方式属性注入
````
1、@Autowired：根据属性类型进行自动装配
  第一步，把service和dao对象创建，在service和dao类添加注解
  第二步，在service注入dao对象，在service类添加dao类型属性，在属性上面使用注解

  @Service
  public class UserServiceImpl implements UserService{
    // 注入属性注解
    @Autowired
    private UserDao userDao;

    @Override
    public void update() {
      System.out.println("service update......");
      userDao.update();
    }
  }
2、@Qualifier：根据属性名称进行注入
  @Qualifier 注解要和 @Autowired 一起使用。
  @Qualifier表示以名称注入，@Autowired表示以类型注入，如果一个接口有多个实现类，那么要用@Qualified指定名称进行注入。

  @Repository(value="userDaoImpl")
  public class UserDaoImpl implements UserDao {

    @Override
    public void update() {
      System.out.println("update......");
    }
  }

  @Service
  public class UserServiceImpl implements UserService{
    // 注入属性注解
    @Autowired
    @Qualifier(value="userDaoImpl")
    private UserDao userDao;
    
    @Override
    public void update() {
      System.out.println("service update......");
    }
  }
3、@Resource：可以根据类型注入，可以根据名称注入
  @Resource：根据类型进行注入
  @Resource(name=“userDaoImpl”)：根据名称进行注入
4、@Value：注入普通类型属性
  @Value(value="abc")
  private String name;
````
# 十、完全注解开发
````
创建配置类,代替xml配置文件
@Configuration //表明该类是一个配置类，相当于xml配置文件
@ComponentScan (basePackages = {"路径名"}) //注解我们想要扫描的包，相当于<context:component-scan base-package="com.example"></context:component-scan>
public class SpringConfig{}
````
````
@Bean
这个注解使用在方法上，用于返回在IOC容器里面加载的类，返回一个值作为容器里面的实例化。

//表示该类是一个配置类
@Configuration
//配置我们想要扫描的包
@ComponentScan("com.example")
public class DataSourceConfig {

    //相当于xml里面bean标签
    @Bean("dataSource")
    public DruidDataSource getDataSource(){
        DruidDataSource dataSource = new DruidDataSource();

        dataSource.setDriverClassName("com.mysql.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost:3306/dbcsdn");
        dataSource.setUsername("root");
        dataSource.setPassword("1234");
        
        return  dataSource;
    }
}
````
````
@PropertySource
加载.properties文件中的配置，相当于<context:property-placeholder location="classpath:druid.properties"></context:property-placeholder>

//表示该类是一个配置类
@Configuration
//配置我们想要扫描的包
@ComponentScan("com.example")
//记载properties文件里面的信息
@PropertySource("classpath:druid.properties")
public class DataSourceConfig {

    @Value("${druid.driver}")
    private String driver;
    @Value("${druid.url}")
    private String url;
    @Value("${druid.username}")
    private String username;
    @Value("${druid.password}")
    private String password;

    //相当于xml里面bean标签
    @Bean("dataSource")
    public DruidDataSource getDataSource(){
        DruidDataSource dataSource = new DruidDataSource();

        dataSource.setDriverClassName(driver);
        dataSource.setUrl(url);
        dataSource.setUsername(username);
        dataSource.setPassword(password);

        return  dataSource;
    }
}
````
````
@Import：用于导入其他的配置类，我们的配置类不可能就只有一个，有一个主的配置类会加载所有的配置类，xml用import标签，注解方式就用Import注解。

// 在主配置文件中引入分配置文件
@Import(DataSourceConfig.class)
public class SpringConfig {}
````