# 一、AOP概念
````
面向切面编程(Aspect Oriented Programming),利用AOP可以对业务逻辑的各个部分进行隔离,使业务逻辑各部分之间的耦合度降低
不修改源码的情况下,添加功能
````
# 二、AOP底层原理
````
AOP底层使用动态代理
1、有接口,使用JDK动态代理
  创建接口实现类代理对象,增强类的方法
2、没有接口,使用CGLIB动态代理
  创建子类的代理对象,增强类的方法
````
# 三、JDK动态代理
````
JDK动态代理的实现：使用Proxy类里面的方法创建代理对象

@Repository(value="userDaoImpl")
public class UserDaoImpl implements UserDao {
  @Override
  public int add(int a, int b) {
    System.out.println("add()方法执行了");
    return a + b;
  }

  @Override
  public String update(String id) {
    return id;
  }
}

调用 newProxyInstance方法，方法有三个参数：
  第一个参数：类加载器
  第二个参数：增强方法所在的类，这个类实现的接口，支持多个接口
  第三个参数：实现这个接口InvocationHandler，创建代理对象，写增强方法

public class JDKProxy {
  public static void main(String[] args) {
    Class[] interface = {UserDao.class};
    UserDaoImpl userDao = new UserDaoImpl();
    UserDao dao = (UserDao) Proxy.newProxyInstance(JDKProxy.class.getClassLoader(), interface, new UserDaoProxy(userDao));
    int result = dao.add(1, 2);
    System.out.println("结果: " + result);
  }
}

class UserDaoProxy implements InvocationHandler {

  // 把需要代理的对象传过来
  private Object obj;
  public UserDaoProxy(Object obj) {
    this.obj = obj;
  }

  @Override
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    //方法执行之前
    System.out.println("方法执行之前: " + method.getName() + " 传递的参数: " + Arrays.toString(args));
    //被增强的方法执行
    Object res = method.invoke(obj, args);
    //方法执行之后
    System.out.println("方法执行之后: " + obj);
    return res;
  }
}

执行结果
方法执行之前: add 传递的参数: [1, 2]
add()方法执行了
方法执行之后: com.jch.spring5.dao.UserDaoImpl@716361
结果: 3
````
# 四、AOP术语
* 1、连接点:可以被增强的方法
* 2、切入点:实际被增强的方法
* 3、通知/增强:实际增强的逻辑称为通知
````
前置通知@Before
后置通知@AfterReturning
环绕通知@Around
异常通知@AfterThrowing
最终通知@After
````
* 4、切面:通知+切入点
````
切入点表达式:execution([权限修饰符] [返回类型][类全路径][方法名称][参数列表])

举例1：对 com.jch.dao.BookDao类里面的add进行增强
execution(* com.jch.dao.BookDao.add(..))

举例2：对 com.jch.dao.BookDao类里面所有的方法进行增强
execution(* com.jch.dao.BookDao.*(..))

举例3：对 com.jch.dao包里面所有的类，类里面所有的方法进行增强
execution(* com.jch.dao.*.*(..))
````
# 五、进行通知的配置
* 1、在Spring配置文件中,开启注解扫描
````
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xmlns:aop="http://www.springframework.org/schema/aop"
        xmlns:tx="http://www.springframework.org/schema/tx"
        xmlns:context="http://www.springframework.org/schema/context"   
        xsi:schemaLocation="
            http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd
            http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-2.5.xsd
            http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-2.5.xsd
            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-3.0.xsd">
````
* 2、使用注解创建User和UserProxy对象
````
// 被增强的类
public class User {
  public void add() {
    System.out.println("add()...");
  }
}
````
* 3、在增强类上面添加注解@Aspect
````
@Component
@Aspect // 生成代理对象
public class UserProxy {
  // 前置通知
  public void before() {
    System.out.println("before()...");
  }
}
````
* 4、在Spring配置文件中开启生成代理
````
在Spring配置文件中开启生成代理对象
如果找到类上有@Aspect注解，将这个类生成代理对象。
<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
````
* 5、AspectJ注解：配置不同类型的通知
````
在增强类的里面，在作为通知方法上面添加通知类型注解，使用切入点表达式配置。

@Component
@Aspect // 生成代理对象
public class UserProxy {
  // 前置通知
  @Before(value="execution(* com.jch.spring5.aop.User.add(...))")
  public void before() {
    System.out.println("before()...");
  }
}
````
````
执行被增强的方法
@Test
public void test() {
  ApplicationContext context = new ClassPathXmlApplicationContext("bean.xml");
  User user = context.getBean("user", User.class);
  user.add();
}

结果
add()...
before()...
````
* 6、配置其他类型通知
````
// 后置通知, 返回值之后执行
@AfterReturning(value="execution(* com.jch.spring5.aop.User.add(...))")
public void afterReturning() {
  System.out.println("afterReturning()...");
}

// 最终通知, 方法执行之后执行
@After(value="execution(* com.jch.spring5.aop.User.add(...))")
public void after() {
  System.out.println("after()...");
}

// 异常通知, 出异常时执行
@AfterThrowing(value="execution(* com.jch.spring5.aop.User.add(...))")
public void afterThrowing() {
  System.out.println("afterThrowing()...");
}

// 环绕通知
@Around(value="execution(* com.jch.spring5.aop.User.add(...))")
public void around() {
  System.out.println("环绕之前...");
  joinPoint.proceed();
  System.out.println("环绕之后...");
}

结果
环绕之前...
before()...
add()...
afterReturning()...
after()...
环绕之后...

有异常时的结果
环绕之前...
before()...
afterThrowing()...
after()...
````
* 7、抽取相同切入点
````
// 相同切入点抽取
@PointCut(value="execution(* com.jch.spring5.aop.User.add(..))")
public void pointDemo() {

}

// 前置通知
@Before(value="pointDemo()")
public void before() {
  System.out.println("before()...");
}
````
* 8、多个增强类对同一个方法进行增强,设置增强优先级
````


@Component
@Aspect // 生成代理对象
@Order(1) // 数值类型
public class PersonProxy {
  // 前置通知
  @Before(value="execution(* com.jch.spring5.aop.User.add(...))")
  public void PersonProxy() {
    System.out.println("PersonProxy()...");
  }
}
````
# 六、完全注解开发
````
@Configuration
@ComponentScan(basePackages = {"com.jch"})
@EnableAspectJAutoProxy(proxyTargetClass = true)
public class ConfigAop {}
````
# 七、配置文件实现AOP
````
<bean id="user" class="com.jch.spring5.aop.User"></bean>
<bean id="userProxy" class="com.jch.spring5.aop.UserProxy"></bean>

<!--配置aop增强-->
<aop:config>
        <!--切入点-->
        <aop:pointcut id="p" expression="execution(* com.jch.spring5.aop.User.add(..))"/>
        <!--配置切面-->
        <aop:aspect ref="userProxy">
                <!--增强作用在具体方法上-->
                <aop:before method="before" pointcut-ref="p"/>
        </aop:aspect>
</aop:config>

````