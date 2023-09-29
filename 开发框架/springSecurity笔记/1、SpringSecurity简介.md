# 一、SpringSecurity 框架简介
````
Spring 是非常流行和成功的 Java 应用开发框架，Spring Security 正是 Spring 家族中的
成员。Spring Security 基于 Spring 框架，提供了一套 Web 应用安全性的完整解决方
案。
正如你可能知道的关于安全方面的两个主要区域是“认证”和“授权”（或者访问控
制），一般来说，Web 应用的安全性包括用户认证（Authentication）和用户授权
（Authorization）两个部分，这两点也是 Spring Security 重要核心功能。
（1）用户认证指的是：验证某个用户是否为系统中的合法主体，也就是说用户能否访问
该系统。用户认证一般要求用户提供用户名和密码。系统通过校验用户名和密码来完成认
证过程。通俗点说就是系统认为用户是否能登录
（2）用户授权指的是验证某个用户是否有权限执行某个操作。在一个系统中，不同用户
所具有的权限是不同的。比如对一个文件来说，有的用户只能进行读取，而有的用户可以
进行修改。一般来说，系统会为不同的用户分配不同的角色，而每个角色则对应一系列的
权限。通俗点讲就是系统判断用户是否有权限去做某些事情。
````
# 二、SpringSecurity 入门案例
* 1、创建一个项
````
加入 Spring Web 和 Spring Security

添加一个配置类
@Configuration
public class SecurityConfigextends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.formLogin() //表单登录
        .and()
        .authorizeRequests() //认证配置
        .anyRequest() //任何请求
        .authenticated(); //都需要身份验证
        }
}
````
* 2、运行这个项目
````
访问 localhost:8080

默认的用户名：user
密码在项目启动的时候在控制台会打印，注意每次启动的时候密码都回发生变化！

输入用户名，密码，这样表示可以访问了，404 表示我们没有这个控制器，但是我们可以
访问了。
````
* 3、权限管理中的相关概念
````
1、主体
英文单词：principal
使用系统的用户或设备或从其他系统远程登录的用户等等。简单说就是谁使用系统谁就是主体。

2、认证
英文单词：authentication
权限管理系统确认一个主体的身份，允许主体进入系统。简单说就是“主体”证明自己是谁。
笼统的认为就是以前所做的登录操作

授权
英文单词：authorization
将操作系统的“权力”“授予”“主体”，这样主体就具备了操作系统中特定功能的能力。
所以简单来说，授权就是给用户分配权限。
````
* 4、添加一个控制器进行访问
````
package com.atguigu.controller;

@Controller
public class IndexController {

    @GetMapping("index")
    @ResponseBody
    public String index(){
        return "success";
    }
}

访问 localhost:8080
````
* 6、UserDetailsService 接口讲解
````
当什么也没有配置的时候，账号和密码是由 Spring Security 定义生成的。而在实际项目中账号和密码都是从数据库中查询出来的。 所以我们要通过自定义逻辑控制认证逻辑。
如果需要自定义逻辑时，只需要实现 UserDetailsService 接口即可。接口定义如下：

public interface UserDetailsService {

    UserDetils loadUserByUsername(String username) throws usernameNotFoundException;
}
 返回值 UserDetails
这个类是系统默认的用户“主体

/ 表示获取登录用户所有权限
Collection<? extends GrantedAuthority> getAuthorities();
// 表示获取密码
String getPassword();
// 表示获取用户名
String getUsername();
// 表示判断账户是否过期
boolean isAccountNonExpired();
// 表示判断账户是否被锁定
boolean isAccountNonLocked();
// 表示凭证{密码}是否过期
boolean isCredentialsNonExpired();
// 表示当前用户是否可用
boolean isEnabled();
````
* 7、PasswordEncoder 接口讲解
````
// 表示把参数按照特定的解析规则进行解析
String encode(CharSequence rawPassword);
// 表示验证从存储中获取的编码密码与编码后提交的原始密码是否匹配。如果密码匹
配，则返回 true；如果不匹配，则返回 false。第一个参数表示需要被解析的密码。第二个
参数表示存储的密码。
boolean matches(CharSequence rawPassword, String encodedPassword);
// 表示如果解析的密码能够再次进行解析且达到更安全的结果则返回 true，否则返回
false。默认返回 false。
default boolean upgradeEncoding(String encodedPassword) {
return false;
}
````
````
接口实现类

BCryptPasswordEncoder 是 Spring Security 官方推荐的密码解析器，平时多使用这个解析器。
BCryptPasswordEncoder 是对 bcrypt 强散列方法的具体实现。是基于 Hash 算法实现的单向加密。可以通过 strength 控制加密强度，默认 10.
````
````
案例

@Test
public void test01(){
    // 创建密码解析器
    BCryptPasswordEncoder bCryptPasswordEncoder = new
    BCryptPasswordEncoder();
    // 对密码进行加密
    String atguigu = bCryptPasswordEncoder.encode("atguigu");
    // 打印加密之后的数据
    System.out.println("加密之后数据：\t"+atguigu);
    // 判断原字符加密后和加密之前是否匹配
    boolean result = bCryptPasswordEncoder.matches("atguigu", atguigu);
    // 打印比较结果
    System.out.println("比较结果：\t"+result);
}
````