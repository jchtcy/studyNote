# 一、@RequestMapping注解
* 1、功能
````
将请求和处理请求的控制器方法关联起来,建立映射关系。
SpringMVC接收到请求,在映射关系中找到对应的控制器来处理请求。
````
* 2、位置
````
标识一个类:类中的所有响应请求的方法都是以该地址作为父路径
标识一个方法:在类的父路径下追加方法上注解中的地址将会访问到该方法

@Controller
@RequestMapping("/test")
public class RequestMappingController {

	//此时请求映射所映射的请求的请求路径为：/test/testRequestMapping
    @RequestMapping("/testRequestMapping")
    public String testRequestMapping(){
        return "success";
    }

}
````
* 3、value属性
````
@RequestMapping注解的value属性通过请求的请求地址匹配请求映射
value属性是一个字符串类型的数组,表示该请求映射能够匹配多个请求地址对应的请求
value属性必须设置,至少通过请求地址匹配请求映射

<a th:href="@{/testRequestMapping}">测试@RequestMapping的value属性-->/testRequestMapping</a><br>
<a th:href="@{/test}">测试@RequestMapping的value属性-->/test</a><br>

@RequestMapping(value = {"/testRequestMapping", "/test"})
public String testRequestMapping(){
    return "success";
}
````
* 4、method属性
````
@RequestMapping注解的method属性通过请求的请求方式(get或post)匹配请求映射
method属性是一个RequestMethod类型的数组,表示该请求映射能够匹配多种请求方式
若当前请求的请求地址满足请求映射的value属性,但请求方式不满足method属性,则浏览器报错405
````
* 5、SpringMVC提供了@RequestMapping的派生注解
````
get请求的映射->GetMapping
post请求的映射->PostMapping
put请求的映射->PutMapping
delete请求的映射->DeleteMapping
````
* 6、params属性
````
@RequestMapping注解的params属性通过请求的请求参数匹配请求映射
"param":要求请求映射所匹配的请求必须携带param请求参数
"!param":要求请求映射所匹配的请求不能携带param请求参数
"param=value":要求请求映射所匹配的请求必须携带param请求参数且param=value
"param!=value":要求请求映射所匹配的请求必须携带param请求参数但param!=value
````
* 7、headers属性
````
@RequestMapping注解的headers属性通过请求的请求头信息匹配请求映射
"header":要求请求映射所匹配的请求必须携带header请求头信息
"!header":要求请求映射所匹配的请求不能携带header请求头信息
"header=value":要求请求映射所匹配的请求必须携带header请求头信息且header=value
"header!=value":要求请求映射所匹配的请求必须携带header请求头信息但header!=value
若当前请求满足@RequestMapping注解的value和method属性,但是不满足headers属性,此时报404错误
````
* 8、SpringMVC支持ant风格的路径
````
?:表示任意的单个字符
*:表示任意的0个或多个字符
**:表示任意的一层或多层目录,
在使用\**时,只能使用/**/xxx的方式
````
* 9、SpringMVC支持路径中的占位符(重点)
````
原始方式:/deleteUser?id=1
rest方式:/deleteUser/1
通过@PathVariable注解实现SpringMVC的restful风格,将某些数据通过路径的方式传输到服务器中

@RequestMapping("/testRest/{id}/{username}")
public String testRest(@PathVariable("id") String id,@PathVariable("username") String username){}
````