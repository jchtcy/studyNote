# 一、HttpMessageConverter
HttpMessageConverter,报文信息转换器。
* 1、@RequestBody
````
@RequestBody可以获取请求体,需要在控制器方法设置一个形参,使用@RequestBody进行标识,当前请求的请求体就会为当前注解所标识的形参赋值。

<form th:action="@{/testRequestBody}" method="post">
    用户名：<input type="text" name="username"><br>
    密码：<input type="password" name="password"><br>
    <input type="submit">
</form>

@RequestMapping("/testRequestBody")
public String testRequestBody(@RequestBody String requestBody){
    System.out.println("requestBody" + requestBody);
    return "success";
}

结果:
requestBody:username=admin&password=123456
````
* 2、RequestEntity
````
RequestEntity封装请求报文的一种类型,需要在控制器方法的形参中设置该类型的形参,当前请求的请求报文就会赋值给该形参,可以通过getHeaders()获取请求头信息,通过getBody()获取请求体信息。

@RequestMapping("/testRequestEntity")
public String testRequestEntity(RequestEntity<String> requestEntity){
    System.out.println("requestHeader" + requestEntity.getHeaders());
    System.out.println("requestBody" + requestEntity.getBody());
    return "success";
}
````
* 3、@ResponseBody
````
@ResponseBody用于标识一个控制器方法,可以将该方法的返回值直接作为相应报文的响应体相应到浏览器

@RequestMapping("/testResponseBody")
@ResponseBody
public String testResponseBody(){
    return "success";
}
````
* 4、SpringMVC处理json
@ResponseBody处理json的步骤：
````
1、导入jackson的依赖
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.12.1</version>
</dependency>

2、在SpringMVC的核心配置文件中开启mvc的注解驱动，此时在HandlerAdaptor中会自动装配一个消息转换器：MappingJackson2HttpMessageConverter，可以将响应到浏览器的Java对象转换为Json格式的字符串
<mvc:annotation-driven />

3、在处理器方法上使用@ResponseBody注解进行标识

4、将Java对象直接作为控制器方法的返回值返回，就会自动转换为Json格式的字符串

@RequestMapping("/testResponseUser")
@ResponseBody
public User testResponseUser(){
    return new User(1001,"admin","123456",23,"男");
}

浏览器的页面中展示的结果：
{“id”:1001,“username”:“admin”,“password”:“123456”,“age”:23,“sex”:“男”}
````
* 5、SpringMVC处理ajax
````
1、请求超链接
<div id="app">
	<a th:href="@{/testAjax}" @click="testAjax">testAjax</a><br>
</div>

2、通过vue和axios处理点击事件
<script type="text/javascript" th:src="@{/static/js/vue.js}"></script>
<script type="text/javascript" th:src="@{/static/js/axios.min.js}"></script>
<script type="text/javascript">
    var vue = new Vue({
        el:"#app",
        methods:{
            testAjax:function (event) {
                axios({
                    method:"post",
                    url:event.target.href,
                    params:{
                        username:"admin",
                        password:"123456"
                    }
                }).then(function (response) {
                    alert(response.data);
                });
                event.preventDefault();
            }
        }
    });
</script>

3、控制器方法
@RequestMapping("/testAjax")
@ResponseBody
public String testAjax(String username, String password){
    System.out.println("username:"+username+",password:"+password);
    return "hello,ajax";
}
````
* 6、@RestController注解
````
@RestController注解是springMVC提供的一个复合注解，标识在控制器的类上，就相当于为类添加了@Controller注解，并且为其中的每个方法添加了@ResponseBody注解
````
* 7、ResponseEntity
````
ResponseEntity用于控制器方法的返回值类型,该控制器方法的返回值就是相应到浏览器的相应报文。
````