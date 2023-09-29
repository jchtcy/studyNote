# 一、日志框架整合
````
Spring5框架自带了通用的日志封装。
Spring5移除了Log4jConfigListener，官方建议使用Log4j2.

第一步，引入依赖
log4j-api-2.11.2
log4j-core-2.11.2
log4j-slf4j-impl-2.11.2
slf4j-api-1.7.30

第二步，创建log4j2.xml(文件名固定)
<?xml version="1.0" encoding="UTF-8"?>
<!--日志级别以及优先级排序: OFF > FATAL > ERROR > WARN > INFO > DEBUG > TRACE > ALL -->
<!--Configuration后面的status用于设置log4j2自身内部的信息输出，可以不设置，当设置成trace时，可以看到log4j2内部各种详细输出-->
<configuration status="INFO">
    <!--先定义所有的appender-->
    <appenders>
        <!--输出日志信息到控制台-->
        <console name="Console" target="SYSTEM_OUT">
            <!--控制日志输出的格式-->
            <PatternLayout pattern="%d{yyyy-MM-dd HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
        </console>
    </appenders>
    <!--然后定义logger，只有定义了logger并引入的appender，appender才会生效-->
    <!--root：用于指定项目的根日志，如果没有单独指定Logger，则会使用root作为默认的日志输出-->
    <loggers>
        <root level="info">
            <appender-ref ref="Console"/>
        </root>
    </loggers>
</configuration>

````
# 二、@Nullable注解
````
@Nullable注解可以使用在方法上面，属性上面，参数上面，表示方法返回可以为空，属性值可以为空，参数值可以为空。

1、注解用在方法上面，方法返回值可以为空。
2、注解用在方法参数里面，方法参数可以为空。
3、注解使用在属性上面，属性值可以为空。
````
# 三、函数式风格GenericApplicationContext
````
Spring5核心容器支持函数式风格创建对象，交给Spring进行管理。

@Test
public void test5() {
    //1 创建GenericApplicationContext对象
    GenericApplicationContext context = new GenericApplicationContext();
    //2 调用context的方法对象注册
    context.refresh();
    context.registerBean("user1", User.class, ()->new User());
    //3 获取在spring注册的对象
    User user = (User) context.getBean("user1");
    System.out.println(user);
}
````
# 三、JUnit
````
JUnit4
第一步:引入spring-test
第二步:创建测试类,使用注解方式完成
    @RunWith(SpringJUnit4ClassRunner.class)//单元测试框架
    @ContextConfiguration("classpath:bean.xml")//加载配置文件
````
````
JUnit5
1、第一步进入JUnit5的jar包
spring-text-5.3.23

2、第二步:创建测试类,使用注解方式完成
@ExtendWith(SpringExtension.class)
@ContextConfiguration("classpath:bean7.xml")
// 或者使用使用一个复合注解替代上面两个注解@SpringJUnitConfig(locations = "classpath:bean.xml")
public class Test5 {

    @Autowired
    private PersonService personService;

    @Test
    public void test() {
        personService.accountMoney();
    }
}

````