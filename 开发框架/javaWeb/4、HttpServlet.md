# 一、引入(我们以后编写的Servlet要继承HttpServlet类)
````
以后我们编写Servlet类的时候，实际上是不会去直接继承GenericServlet类的，因为我们是B/S结构的系统，这种系统是基于HTTP超文本传输协议的，在Servlet规范当中，提供了一个类叫做HttpServlet，它是专门为HTTP协议准备的一个Servlet类。我们编写的Servlet类要继承HttpServlet。（HttpServlet是HTTP协议专用的。）使用HttpServlet处理HTTP协议更便捷。但是你需要直到它的继承结构：

javax.servlet.Servlet（接口）【爷爷】
javax.servlet.GenericServlet implements Servlet（抽象类）【儿子】
javax.servlet.http.HttpServlet extends GenericServlet（抽象类）【孙子
````
# 二、HttpServlet源码分析
* 1、执行流程
````
package com.jch.javaweb;

import javax.servlet.http.HttpServlet;

public class HelloServlet extends HttpServlet {
    /**1、调用HttpServlet的无参构造方法
     * 2、调用GenericServlet的有参数init(ServletConfig config)
     * 3、调用GenericServlet的无参数init()
     * 4、调用HttpServlet的service(ServletRequest req, ServletResponse res)
     *      这一步会将将ServletRequest和ServletResponse向下转型为带有Http的HttpServletRequest和HttpServletResponse
     *      调用重载的service方法 service(request, response);
     * 5、调用HttpServlet的 service(HttpServletRequest req, HttpServletResponse resp)
     */

}
````
* 2、Servlet类的开发步骤：
````
第一步:编写一个Servlet类，直接继承HttpServlet
第二步:重写doGet方法或者重写doPost方法，到底重写谁，javaweb程序员说了算。
第三步:将Servlet类配置到web.xml文件当中。
第四步:准备前端的页面（form表单），form表单中指定请求路径即可
````
# 三、web站点的欢迎页面
````
如果我们访问的方式是：
http://localhost:8080/servlet05 如果我们访问的就是这个站点，没有指定具体的资源路径。它默认会访问谁呢？
默认会访问你设置的欢迎页面。

创建/welcome/login.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>login page</title>
</head>
<body>
<h1>login page</h1>
</body>
</html>

在web.xml文件中配置
<welcome-file-list>
    <welcome-file>welcome/login.html</welcome-file>
    <!--路径不需要以“/”开始，并且路径默认从webapp的根下开始找。-->
</welcome-file-list>
````
````
一个webapp是可以设置多个欢迎页面的
<welcome-file-list>
    <welcome-file>page1/page2/page.html</welcome-file>
    <welcome-file>login.html</welcome-file>
</welcome-file-list>
````
````
实际上配置欢迎页面有两个地方可以配置：
一个是在webapp内部的web.xml文件中。（在这个地方配置的属于局部配置）
一个是在CATALINA_HOME/conf/web.xml文件中进行配置。（在这个地方配置的属于全局配置）
Tomcat服务器的全局欢迎页面是：index.html index.htm index.jsp。
如果你一个web站点没有设置局部的欢迎页面，Tomcat服务器就会以index.html index.htm index.jsp作为一个web站点的欢迎页面。
````
````
欢迎页面也可以是一个servlet
    <servlet>
        <servlet-name>welcomeServlet</servlet-name>
        <servlet-class>com.bjpowernode.javaweb.servlet.WelcomeServlet</servlet-class>
    </servlet>
    <servlet-mapping>
        <servlet-name>welcomeServlet</servlet-name>
        <url-pattern>/welcome</url-pattern>
    </servlet-mapping>
    <welcome-file-list>
        <welcome-file>welcome</welcome-file>
    </welcome-file-list>
````
# 四、关于WEB-INF目录
````
在WEB-INF目录下新建了一个文件：welcome.html
打开浏览器访问：http://localhost:8080/servlet07/WEB-INF/welcome.html 出现了404错误。
注意：放在WEB-INF目录下的资源是受保护的。在浏览器上不能够通过路径直接访问。所以像HTML、CSS、JS、image等静态资源一定要放到WEB-INF目录之外
````
# 五、HttpServletRequest接口详解
* 1、源码分析
````
1、HttpServletRequest是一个接口，全限定名称：javax.servlet.http.HttpServletRequest
2、HttpServletRequest接口是Servlet规范中的一员。
3、HttpServletRequest接口的父接口：ServletRequest
    public interface HttpServletRequest extends ServletRequest {}
4、org.apache.catalina.connector.RequestFacade 实现了 HttpServletRequest接口
    public class RequestFacade implements HttpServletRequest {}
5、Tomcat服务器（WEB服务器、WEB容器）实现了HttpServletRequest接口，还是说明了Tomcat服务器实现了Servlet规范。而对于我们javaweb程序员来说，实际上不需要关心这个，我们只需要面向接口编程即可。我们关心的是HttpServletRequest接口中有哪些方法，这些方法可以完成什么功能！！！！
````
* 2、HttpServletRequest对象中的信息
````
封装了HTTP的请求协议。
实际上是用户发送请求的时候，遵循了HTTP协议，发送的是HTTP的请求协议，Tomcat服务器将HTTP协议中的信息以及数据全部解析出来，然后Tomcat服务器把这些信息封装到HttpServletRequest对象当中，传给了我们javaweb程序员。
javaweb程序员面向HttpServletRequest接口编程，调用方法就可以获取到请求的信息了。
````
* 3、request和response对象的生命周期
````
request对象和response对象，一个是请求对象，一个是响应对象。这两个对象只在当前请求中有效。
一次请求对应一个request。
````
* 4、HttpServletRequest接口中常用的方法
````
Map<String,String[]> getParameterMap() 这个是获取Map
Enumeration<String> getParameterNames() 这个是获取Map集合中所有的key
String[] getParameterValues(String name) 根据key获取Map集合的value
String getParameter(String name)  根据key获取value这个一维数组当中的第一个元素。这个方法最常用。
````
* 5、应用域对象
````
ServletContext （Servlet上下文对象。）

什么情况下会考虑向ServletContext这个应用域当中绑定数据呢
    第一：所有用户共享的数据。
    第二：这个共享的数据量很小。
    第三：这个共享的数据很少的修改操作。
    在以上三个条件都满足的情况下，使用这个应用域对象，可以大大提高我们程序执行效率。
    实际上向应用域当中绑定数据，就相当于把数据放到了缓存（Cache）当中，然后用户访问的时候直接从缓存中取，减少IO的操作，大大提升系统的性能，所以缓存技术是提高系统性能的重要手段。
你见过哪些缓存技术呢？
    字符串常量池
    整数型常量池 [-128~127]，但凡是在这个范围当中的Integer对象不再创建新对象，直接从这个整数型常量池中获取。大大提升系统性能。
    数据库连接池（提前创建好N个连接对象，将连接对象放到集合当中，使用连接对象的时候，直接从缓存中拿。省去了连接对象的创建过程。效率提升。）
    线程池（Tomcat服务器就是支持多线程的。所谓的线程池就是提前先创建好N个线程对象，将线程对象存储到集合中，然后用户请求过来之后，直接从线程池中获取线程对象，直接拿来用。提升系统性能）
    后期你还会学习更多的缓存技术，例如：redis、mongoDB…
ServletContext当中有三个操作域的方法：
    void setAttribute(String name, Object obj); // 向域当中绑定数据。
    Object getAttribute(String name); // 从域当中根据name获取数据。
    void removeAttribute(String name); // 将域当中绑定的数据移除
````
* 6、“请求域”对象
````
“请求域”对象要比“应用域”对象范围小很多。生命周期短很多。请求域只在一次请求内有效。
一个请求对象request对应一个请求域对象。一次请求结束之后，这个请求域就销毁了。

请求域对象也有这三个方法：
    void setAttribute(String name, Object obj); // 向域当中绑定数据。
    Object getAttribute(String name); // 从域当中根据name获取数据。
    void removeAttribute(String name); // 将域当中绑定的数据移除
请求域和应用域的选用原则？
    尽量使用小的域对象，因为小的域对象占用的资源较少。
````
* 7、转发
````
// 第一步：获取请求转发器对象
// 相当于把"/b"这个路径包装到请求转发器当中，实际上是把下一个跳转的资源的路径告知给Tomcat服务器了。
RequestDispatcher dispatcher = request.getRequestDispatcher("/b");

// 第二步：调用请求转发器RequestDispatcher的forward方法。进行转发。
// 转发的时候：这两个参数很重要。request和response都是要传递给下一个资源的。
dispatcher.forward(request, response);
````
````
两个Servlet怎么共享数据？
将数据放到ServletContext应用域当中，当然是可以的，但是应用域范围太大，占用资源太多。不建议使用。
可以将数据放到request域当中，然后AServlet转发到BServlet，保证AServlet和BServlet在同一次请求当中，这样就可以做到两个Servlet，或者多个Servlet共享同一份数据。
````
````
转发的下一个资源必须是一个Servlet吗？
不一定，只要是Tomcat服务器当中的合法资源，都是可以转发的。例如：html…
注意：转发的时候，路径的写法要注意，转发的路径以“/”开始，不加项目名。
// 转发到一个Servlet，也可以转发到一个HTML，只要是WEB容器当中的合法资源即可。
request.getRequestDispatcher("/test.html").forward(request, response);
````
* 8、request对象中两个非常容易混淆的方法
````
// uri?username=zhangsan&userpwd=123&sex=1
String username = request.getParameter("username");

// 之前一定是执行过：request.setAttribute("name", new Object())
Object obj = request.getAttribute("name");

// 以上两个方法的区别是什么？
// 第一个方法：获取的是用户在浏览器上提交的数据。
// 第二个方法：获取的是请求域当中绑定的数据。
````
* 9、HttpServletRequest接口的其他常用方法
````
// 获取客户端的IP地址
String remoteAddr = request.getRemoteAddr();
// request.getRemoteHost(); 获取远程主机地址
// request.getRemotePort(); 获取端口

// get请求在请求行上提交数据。
// post请求在请求体中提交数据。
// 设置请求体的字符集。（显然这个方法是处理POST请求的乱码问题。这种方式并不能解决get请求的乱码问题。）
// Tomcat10之后，request请求体当中的字符集默认就是UTF-8，不需要设置字符集，不会出现乱码问题。
// Tomcat9前（包括9在内），如果前端请求体提交的是中文，后端获取之后出现乱码，怎么解决这个乱码？执行以下代码。
request.setCharacterEncoding("UTF-8");

// 在Tomcat9之前（包括9），响应中文也是有乱码的，怎么解决这个响应的乱码？
response.setContentType("text/html;charset=UTF-8");
// 在Tomcat10之后，包括10在内，响应中文的时候就不在出现乱码问题了。以上代码就不需要设置UTF-8了。

// 注意一个细节
// 在Tomcat10包括10在内之后的版本，中文将不再出现乱码。（这也体现了中文地位的提升。）

// get请求乱码问题怎么解决？
// get请求发送的时候，数据是在请求行上提交的，不是在请求体当中提交的。
// get请求乱码怎么解决
// 方案：修改CATALINA_HOME/conf/server.xml配置文件
<Connector URIEncoding="UTF-8" />
// 注意：从Tomcat8之后，URIEncoding的默认值就是UTF-8，所以GET请求也没有乱码问题了。
    
// 获取应用的根路径
String contextPath = request.getContextPath();

// 获取请求方式
String method = request.getMethod();

// 获取请求的URI
String uri = request.getRequestURI();  // /aaa/testRequest

// 获取servlet path，不带项目名
String servletPath = request.getServletPath(); //   /testRequest
````
# 六、Servlet注解，简化配置
````
@WebServlet(name = "hello",
        urlPatterns = {"/hello1", "/hello2", "/hello3"},
        loadOnStartup = 1, // 服务器部署启动时加载该类
        // 初始化参数
 initParams = {@WebInitParam(name="username", value="root"), @WebInitParam(name="password", value="123")})
public class HelloServlet extends HttpServlet {

    // 无参数构造方法

    public HelloServlet() {
        System.out.println("无参数构造方法执行，HelloServlet加载完成");
    }


    /*@WebServlet
    private String name;*/

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        response.setContentType("text/html;charset=UTF-8");
        PrintWriter out = response.getWriter();
        // 获取Servlet Name
        String servletName = getServletName();
        out.print("servlet name = " + servletName + "<br>");

        // 获取servlet path
        String servletPath = request.getServletPath();
        out.print("servlet path = " + servletPath + "<br>");

        // 获取初始化参数
        Enumeration<String> names = getInitParameterNames();
        while (names.hasMoreElements()) {
            String name = names.nextElement();
            String value = getInitParameter(name);
            out.print(name + "=" + value + "<br>");
        }
    }
}
````
# 七、通过反射获取注解
````
@WebServlet("/wel")
public class WelcomeServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        response.setContentType("text/html;charset=UTF-8");
        PrintWriter out = response.getWriter();
        out.print("欢迎学习Servlet。");
    }
}

public class ReflectAnnotation {
    public static void main(String[] args) throws Exception{

        // 使用反射机制将类上面的注解进行解析。
        // 获取类Class对象
        Class<?> welcomeServletClass = Class.forName("com.jch.javaweb.servlet.WelcomeServlet");

        // 获取这个类上面的注解对象
        // 先判断这个类上面有没有这个注解对象，如果有这个注解对象，就获取该注解对象。
        //boolean annotationPresent = welcomeServletClass.isAnnotationPresent(WebServlet.class);
        //System.out.println(annotationPresent);
        if (welcomeServletClass.isAnnotationPresent(WebServlet.class)) {
            // 获取这个类上面的注解对象
            WebServlet webServletAnnotation = welcomeServletClass.getAnnotation(WebServlet.class);
            // 获取注解的value属性值。
            String[] value = webServletAnnotation.value();
            for (int i = 0; i < value.length; i++) {
                System.out.println(value[i]);
            }
        }

    }
}
````