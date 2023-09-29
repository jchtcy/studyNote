# 一、SpringMVC的视图
* 1、ThymeleafView
````
当控制器方法设置的视图名称没有前缀时,此视图名称会被SpringMVC配置文件中所配置的视图解析器解析,视图名称拼接视图前缀和视图后缀得到最终路径,通过转发的方式实现跳转

@RequestMapping("/testThymeleafView")
public String testHello(){
    return "success";
}
````
* 2、转发视图
````
当控制器方法中设置的视图名以"forward:"为前缀时,创建InternalResourceView视图,此时的视图名称不会被SpringMVC配置文件中所配置的视图解析器解析,而是会将前缀"forward:"去掉,剩余部分作为最终路径通过转发的方式实现跳转。

@RequestMapping("/testForward")
public String testForward(){
    return "forward:/testThymeleafView";
}
````
* 3、重定向视图
````
当控制器方法中所设置的视图名称以"redirect:"为前缀时,创建RedirectView视图,此时的视图名称不会被SpringMVC配置文件中所配置的视图解析器解析,而是会将前缀"redirect:"去掉,剩余部分作为最终路径通过重定向的方式实现跳转。

@RequestMapping("/testRedirect")
public String testRedirect(){
    return "redirect:/testThymeleafView";
}
````
* 4、视图控制器view-controller
````
当控制器方法中,仅仅用来实现页面跳转,即只需设置视图名称时,可以将处理器方法使用view-controller标签进行表示。

<!--
	path：设置处理的请求地址
	view-name：设置请求地址所对应的视图名称
-->
<mvc:view-controller path="/testView" view-name="success"></mvc:view-controller>

注：

当SpringMVC中设置任何一个view-controller时，其他控制器中的请求映射将全部失效，此时需要在SpringMVC的核心配置文件中设置开启mvc注解驱动的标签：
<mvc:annotation-driven />
````