# 一、Filter
* 1、定义
````
Filter是过滤器。
Filter可以在Servlet这个目标程序执行之前添加代码。也可以在目标Servlet执行之后添加代码。之前之后都可以添加过滤规则。
一般情况下，都是在过滤器当中编写公共代码。
````
* 2、过滤器怎么写
````
第一步：编写一个Java类实现一个接口：javax.servlet.Filter。并且实现这个接口当中所有的方法。
    init方法：在Filter对象第一次被创建之后调用，并且只调用一次。
    doFilter方法：只要用户发送一次请求，则执行一次。发送N次请求，则执行N次。在这个方法中编写过滤规则。
    destroy方法：在Filter对象被释放/销毁之前调用，并且只调用一次。
第二步：在web.xml文件中对Filter进行配置。这个配置和Servlet很像
    <filter>
        <filter-name>filter2</filter-name>
        <filter-class>com.bjpowernode.javaweb.servlet.Filter2</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>filter2</filter-name>
        <url-pattern>*.do</url-pattern>
    </filter-mapping>
    或者使用注解：@WebFilter({“*.do”})
````
* 3、目标Servlet是否执行，取决于两个条件：
````
第一：在过滤器当中是否编写了：chain.doFilter(request, response); 代码
第二：用户发送的请求路径是否和Servlet的请求路径一致。
````
* 4、注意
````
chain.doFilter(request, response); 这行代码的作用：
    执行下一个过滤器，如果下面没有过滤器了，执行最终的Servlet。
注意：Filter的优先级，天生的就比Servlet优先级高。
    /a.do 对应一个Filter，也对应一个Servlet。那么一定是先执行Filter，然后再执行Servlet。
在web.xml文件中进行配置的时候，Filter的执行顺序是什么？
    依靠filter-mapping标签的配置位置，越靠上优先级越高
使用@WebFilter的时候，Filter的执行顺序是怎样的呢？
    执行顺序是：比较Filter这个类名。
    比如：FilterA和FilterB，则先执行FilterA。
    比如：Filter1和Filter2，则先执行Filter1.
Filter的生命周期？
    和Servlet对象生命周期一致。
    唯一的区别：Filter默认情况下，在服务器启动阶段就实例化。Servlet不会。

````