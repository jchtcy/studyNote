# 一、JSP概述
* 1、JSP实际上就是一个Servlet。
````
index.jsp访问的时候，会自动翻译生成index_jsp.java，会自动编译生成index_jsp.class，那么index_jsp 这就是一个类。
index_jsp 类继承 HttpJspBase，而HttpJspBase类继承的是HttpServlet。所以index_jsp类就是一个Servlet类。
jsp的生命周期和Servlet的生命周期完全相同。完全就是一个东西。没有任何区别。
jsp和servlet一样，都是单例的。（假单例。）
````
* 2、jsp文件第一次访问的时候是比较慢的，为什么？
````
为什么大部分的运维人员在给客户演示项目的时候，为什么提前先把所有的jsp文件先访问一遍。
第一次比较麻烦：
要把jsp文件翻译生成java源文件
java源文件要编译生成class字节码文件
然后通过class去创建servlet对象
然后调用servlet对象的init方法
最后调用servlet对象的service方法。
第二次就比较快了，为什么？
因为第二次直接调用单例servlet对象的service方法即可
````
* 3、JSP是什么？
````
JSP是java程序。（JSP本质还是一个Servlet）
JSP是：JavaServer Pages的缩写。（基于Java语言实现的服务器端的页面。）
Servlet是JavaEE的13个子规范之一，那么JSP也是JavaEE的13个子规范之一。
JSP是一套规范。所有的web容器/web服务器都是遵循这套规范的，都是按照这套规范进行的“翻译”
每一个web容器/web服务器都会内置一个JSP翻译引擎。
````
* 4、JSP既然本质上是一个Servlet，那么JSP和Servlet到底有什么区别呢？
````
职责不同：
Servlet的职责是什么：收集数据。（Servlet的强项是逻辑处理，业务处理，然后链接数据库，获取/收集数据。）
JSP的职责是什么：展示数据。（JSP的强项是做数据的展示）
````
# 二、JSP的基础语法
* 1、JSP中直接编写文字
````
在jsp文件中直接编写文字，都会自动被翻译到哪里？

翻译到servlet类的service方法的out.write(“翻译到这里”)，直接翻译到双引号里，被java程序当做普通字符串打印输出到浏览器。
在JSP中编写的HTML CSS JS代码，这些代码对于JSP来说只是一个普通的字符串。但是JSP把这个普通的字符串一旦输出到浏览器，浏览器就会对HTML CSS JS进行解释执行。展现一个效果。
````
* 2、JSP中编写java程序
````
<% java语句; %>

<%
  System.out.println("hello JSP");
%>
````
* 3、JSP的输出语句 <%= %>
````
<%= %> 翻译成了这个java代码: out.print();

<%="hello world"%>
````
* 4、JSP中的注释
````
<%--JSP的专业注释，不会被翻译到java源代码当中。--%>
````
# 三、page指令当中都有哪些常用的属性
````
<%@page session="true|false" %>
true表示启用JSP的内置对象session，表示一定启动session对象。没有session对象会创建。
如果没有设置，默认值就是session="true"
session="false" 表示不启动内置对象session。当前JSP页面中无法使用内置对象session。

<%@page contentType="text/json" %>
contentType属性用来设置响应的内容类型
但同时也可以设置字符集。
默认 text/html
<%@page contentType="text/json;charset=UTF-8" %>

<%@page pageEncoding="UTF-8" %>
pageEncoding="UTF-8" 表示设置响应时采用的字符集。
设置了<%@page contentType="text/json;charset=UTF-8" %>，又使用pageEncoding，以contentType="text/json;charset=UTF-8"为准

<%@page import="java.util.List, java.util.Date, java.util.ArrayList" %>
<%@page import="java.util.*" %>
import语句，导包。

<%@page errorPage="/error.jsp" %>
当前页面出现异常之后，跳转到error.jsp页面。
errorPage属性用来指定出错之后的跳转位置。

<%@page isErrorPage="true" %>
表示启用JSP九大内置对象之一：exception
默认值是false。
配置的页面可以使用异常对象exceptio
````
# 四、JSP的九大内置对象
````
javax.servlet.jsp.PageContext pageContext 页面作用域
javax.servlet.http.HttpServletRequest request 请求作用域，一次请求可以跨多页页面
javax.servlet.http.HttpSession session 会话作用域
javax.servlet.ServletContext application 应用作用域
    pageContext < request < session < application
    以上四个作用域都有：setAttribute、getAttribute、removeAttribute方法。
    以上作用域的使用原则：尽可能使用小的域。
java.lang.Throwable exception
javax.servlet.ServletConfig config
java.lang.Object page （其实是this，当前的servlet对象）
javax.servlet.jsp.JspWriter out （负责输出）
javax.servlet.http.HttpServletResponse response （负责响应）
````
# 五、EL表达式的使用(${表达式})
* 1、从域中取数据
````
<%
  // 创建User对象
  User user = new User();
  user.setUsername("jackson");
  user.setPassword("1234");
  user.setAge(50);

  // 将User对象存储到某个域当中。一定要存，因为EL表达式只能从某个范围中取数据。
  // 数据是必须存储到四大范围之一的。
  request.setAttribute("userObj", user);
%>

<%--使用EL表达式取--%>
${这个位置写什么？？？？这里写的一定是存储到域对象当中时的name}
要这样写：
${userObj}
等同于java代码：<%=request.getAttribute("userObj")%>
你不要这样写：${"userObj"}
````
* 2、EL表达式，怎么从Map集合中取数据
````
${map.key}

<%@ page import="java.util.HashMap" %>
<%@ page import="java.util.Map" %>
<%@ page import="com.bjpowernode.javaweb.jsp.bean.User" %>
<%@page contentType="text/html; charset=UTF-8" %>

<%
    // 一个Map集合
    Map<String,String> map = new HashMap<>();
    map.put("username", "zhangsan");
    map.put("password", "123");

    // 将Map集合存储到request域当中。
    request.setAttribute("userMap", map);

    Map<String,User> userMap2 = new HashMap<>();
    User user = new User();
    user.setUsername("lisi");
    userMap2.put("user", user);
    request.setAttribute("fdsafdsa", userMap2);
%>

<%--使用EL表达式将Map集合中的user对象中的username取出--%>
${fdsafdsa.user.username}

<hr>

<%--使用EL表达式，将map中的数据取出，并输出到浏览器--%>
${userMap.username}
<br>
${userMap.password}
<br>
${userMap["username"]}
<br>
${userMap["password"]}
````
* 3、EL表达式，怎么从数组和List集合中取数据：
````
${数组[0]}
${数组[1]}
${list[0]}

<%
    // 数组对象
    String[] usernames = {"zhangsan", "lisi", "wangwu"};

    // 向域中存储数组
    request.setAttribute("nameArray", usernames);

    User u1 = new User();
    u1.setUsername("zhangsan");

    User u2 = new User();
    u2.setUsername("lisi");

    User[] users = {u1, u2};

    request.setAttribute("userArray", users);

    List<String> list = new ArrayList<>();
    list.add("abc");
    list.add("def");

    request.setAttribute("myList", list);

    Set<String> set = new HashSet<>();
    set.add("a");
    set.add("b");

    request.setAttribute("set", set);
%>

<%--取出set集合--%>
set集合：${set}
<%--无法获取：PropertyNotFoundException: 类型[java.util.HashSet]上找不到属性[0]--%>
<%-- set无序，没有索引 --%>
<%--${set[0]}--%>
<hr>

<%--取出List集合--%>
<%--list集合也是通过下标的方式取数据的。--%>
${myList}
${myList[0]}
${myList[1]}
<hr>

<%--取出数组中第二个用户的用户名--%>
${userArray[1].username}
<hr>

<%--使用EL表达式取出数组中的元素--%>
${nameArray} <%--将数组对象直接输出--%>
${nameArray[0]} <%--取出数组中的第一个元素--%>
${nameArray[1]}
${nameArray[2]}

<%--取不出数据，在浏览器上直接显示的就是空白。不会出现数组下表越界。--%>
${nameArray[100]}
````
