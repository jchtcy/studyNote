# 一、Listener监听器
* 1、什么是监听器？
````
监听器是Servlet规范中的一员。就像Filter一样。Filter也是Servlet规范中的一员。
在Servlet中，所有的监听器接口都是以“Listener”结尾。
````
* 2、监听器有什么用？
````
监听器实际上是Servlet规范留给我们javaweb程序员的特殊时机。
特殊的时刻如果想执行这段代码，你需要想到使用对应的监听器。
````
* 3、Servlet规范中提供了哪些监听器？
````
javax.servlet包下：
    1、ServletContextListener
    2、ServletContextAttributeListener
    3、ServletRequestListener
    4、ServletRequestAttributeListener
javax.servlet.http包下：
    1、HttpSessionListener
    2、HttpSessionAttributeListener
        该监听器需要使用@WebListener注解进行标注。
        该监听器监听的是什么？是session域中数据的变化。只要数据变化，则执行相应的方法。主要监测点在session域对象上。
        @WebListener
        public class MyHttpSessionAttributeListener implements HttpSessionAttributeListener {

            // 向session域当中存储数据的时候，以下方法被WEB服务器调用。
            @Override
            public void attributeAdded(HttpSessionBindingEvent se) {
                System.out.println("session data add");
            }

            // 将session域当中存储的数据删除的时候，以下方法被WEB服务器调用。
            @Override
            public void attributeRemoved(HttpSessionBindingEvent se) {
                System.out.println("session data remove");
            }

            // session域当中的某个数据被替换的时候，以下方法被WEB服务器调用。
            @Override
            public void attributeReplaced(HttpSessionBindingEvent se) {
                System.out.println("session data replace");
            }
        }
    3、HttpSessionBindingListener
        该监听器不需要使用@WebListener进行标注。
        假设User类实现了该监听器，那么User对象在被放入session的时候触发bind事件，User对象从session中删除的时候，触发unbind事件。
        假设Customer类没有实现该监听器，那么Customer对象放入session或者从session删除的时候，不会触发bind和unbind事件。
        public class User1 implements HttpSessionBindingListener {

            @Override
            public void valueBound(HttpSessionBindingEvent event) {
                System.out.println("绑定数据");
            }

            @Override
            public void valueUnbound(HttpSessionBindingEvent event) {
                System.out.println("解绑数据");
            }
        }
    4、HttpSessionIdListener
        session的id发生改变的时候，监听器中的唯一一个方法就会被调用。
    5、HttpSessionActivationListener
        监听session对象的钝化和活化的。
        钝化：session对象从内存存储到硬盘文件。
        活化：从硬盘文件把session恢复到内存。

````
* 4、实现一个监听器的步骤：以ServletContextListener为例
````
第一步：编写一个类实现ServletContextListener接口。并且实现里面的方法。
    在ServletContext对象被创建的时候调用
    void contextInitialized(ServletContextEvent event)
    在ServletContext对象被销毁的时候调用
    void contextDestroyed(ServletContextEvent event)
第二步：在web.xml文件中对ServletContextListener进行配置
    <listener>
        <listener-class>com.bjpowernode.javaweb.listener.MyServletContextListener</listener-class>
    </listener>
````