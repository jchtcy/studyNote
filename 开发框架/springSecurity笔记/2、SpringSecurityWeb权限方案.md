# 一、SpringSecurity Web 权限方案
* 1、设置登录系统的账号、密码
````
方式一：在 application.properties
spring.security.user.name=admin
spring.security.user.password=123456

方式二：编写类实现接口
package com.jch.config;
@Configuration
public class SecurityConfig {
    //注入 PasswordEncoder类到 spring容器中
    @Bean
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }
}

package com.jch.service;
@Service
public class LoginService implements UserDetailsService {
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // 判断用户名是否存在
        if (!"admin".equals(username)){
        throw new UsernameNotFoundException("用户名不存在！");
        }
        // 从数据库中获取的密码 admin的密文
        String pwd = "$2a$10$2R/M6iU3mCZt3ByG7kwYTeeW0w7/UqdeXrb27zkBIizBvAven0/na";
        // 第三个参数表示权限
        return new User(username,pwd,AuthorityUtils.commaSeparatedStringToAuthorityList("admin,"));
    }
}
````
* 2、实现数据库认证来完成用户登录
````
1、添加依赖
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <!--mybatis-plus-->
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.0.5</version>
    </dependency>
    <!--mysql-->
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
    <!--lombok 用来简化实体类-->
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
</dependencies>
````
* 3、实体类
````
@Data
public class Users {
    private Integer id;
    private String username;
    private String password;
}
````
* 4、整合 MybatisPlus 制作 mapper
````
@Repository
public interface UsersMapper extends BaseMapper<Users> {}

配置文件添加数据库配置
#mysql 数据库连接
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/demo?serverTimezone=GMT%2B8
spring.datasource.username=root
spring.datasource.password=root
````
* 5、制作登录实现类
````
@Service("userDetailsService")
public class MyUserDetailsService implements UserDetailsService {
    @Autowired
    private UsersMapper usersMapper;
    @Override
    public UserDetails loadUserByUsername(String s) throws UsernameNotFoundException {
        QueryWrapper<Users> wrapper = new QueryWrapper();
        wrapper.eq("username",s);
        Users users = usersMapper.selectOne(wrapper);
        if(users == null) {
            throw new UsernameNotFoundException("用户名不存在！");
        }
        List<GrantedAuthority> auths = AuthorityUtils.commaSeparatedStringToAuthorityList("role");
        return new User(users.getUsername(),new BCryptPasswordEncoder().encode(users.getPassword()),auths);
    }
}
````
* 6、未认证请求跳转到登录页
````
1、引入前端模板依赖
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>

2、编写登录界面

3、编写控制器
@Controller
public class IndexController {

    @GetMapping("index")
    public String index(){
        return "login";
    }

    @GetMapping("findAll")
    @ResponseBody
    public String findAll(){
        return "findAll";
    }
}

4、编写配置类放行登录页面以及静态资源

@Configuration
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    //注入 PasswordEncoder类到 spring容器中
    @Bean
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
        .antMatchers("/layui/**","/index") //表示配置请求路径
        .permitAll() //指定 URL无需保护。
        .anyRequest() //其他请求
        .authenticated(); //需要认证
    }
}

5、设置未授权的请求跳转到登录页
配置类
@Override
protected void configure(HttpSecurity http) throws Exception {
    //配置认证
    http.formLogin().loginPage("/index") //配置哪个 url为登录页面
        .loginProcessingUrl("/login") //设置哪个是登录的 url
        .successForwardUrl("/success") //登录成功之后跳转到哪个 url
        .failureForwardUrl("/fail");//登录失败之后跳转到哪个 url

    http.authorizeRequests()
        .antMatchers("/layui/**","/index") //表示配置请求路径
        .permitAll() //指定 URL无需保护。
        .anyRequest() //其他请求
        .authenticated(); //需要认证
    //关闭 csrf
    http.csrf().disable();
}

控制器
@PostMapping("/success")
public String success(){
    return "success";
}
@PostMapping("/fail")
public String fail(){
    return "fail";
}

<form action="/login"method="post">
用户名:<input type="text"name="username"/><br/>
密码：<input type="password"name="password"/><br/>
<input type="submit"value="提交"/>
</form>
注
页面提交方式必须为 post 请求，所以上面的页面不能使用，用户名，密码必须为
username,password
````
# 二、基于角色或权限进行访问控制
* 1、hasAuthority 方法
````
如果当前的主体具有指定的权限，则返回 true,否则返回 false

修改配置类
    http.authorizeRequests()
        .antMatchers("/layui/**","/index") //表示配置请求路径
        .permitAll() //指定 URL无需保护
        .antMarchers("/findAll").hasAuthority("admin")//用户是否有admin权限
        .antMarchers("/find").hasAuthority("role")//用户是否有role权限
        .anyRequest() //其他请求
        .authenticated(); //需要认证
    //关闭 csrf
    http.csrf().disable();

添加一个控制器
@GetMapping("/find")
@ResponseBody
public String find() {
    return "find";
}   

给用户登录主体赋予权限
return new User(username,pwd,AuthorityUtils.commaSeparatedStringToAuthorityList("admin,role"));
````
* 2、hasAnyAuthority 方法
````
如果当前的主体有任何提供的角色（给定的作为一个逗号分隔的字符串列表）的话，返回true.
````
* 3、hasRole 方法
````
如果用户具备给定角色就允许访问,否则出现 403。
如果当前主体具有指定的角色，则返回 true。

底层源码：
private static String hasRole(String role) {
    Assert.notNull(role, "role cannot be null");
    if (role.startWith("ROLE_")) {
        throw new IllegalArgumentException("role should not start with 'ROLE_' since it is automatically inserted.");
    } else {
        return "hasRole('ROLE_" + ROLE + "')";
    }
}

给用户添加角色
return new User(username,pwd,AuthorityUtils.commaSeparatedStringToAuthorityList("admin,role", "ROLE_admin"));

修改配置文件：
注意配置文件中不需要添加”ROLE_“，因为上述的底层代码会自动添加与之进行匹配
    http.authorizeRequests()
        .antMatchers("/layui/**","/index") //表示配置请求路径
        .permitAll() //指定 URL无需保护
        .antMarchers("/findAll").hasRole("admin")//用户是否有admin权限
        .anyRequest() //其他请求
        .authenticated(); //需要认证
    //关闭 csrf
    http.csrf().disable();
````
* 4、hasAnyRole
````
表示用户具备任何一个条件都可以访问。

给用户添加角色
return new User(username,pwd,AuthorityUtils.commaSeparatedStringToAuthorityList("admin,role", "ROLE_admin", "ROLE_role"));

修改配置文件
    http.authorizeRequests()
        .antMatchers("/layui/**","/index") //表示配置请求路径
        .permitAll() //指定 URL无需保护
        .antMarchers("/findAll").hasRole("admin")//用户是否有admin权限
        .antMarchers("/findAll").hasAnyRole("role", "role1")//用户是否有role或role1权限
        .anyRequest() //其他请求
        .authenticated(); //需要认证
    //关闭 csrf
    http.csrf().disable();
````
* 5、自定义 403 页面
````
1、修改访问配置类
http.exceptionHandling().accessDeniedPage("/unauth");

2、添加对应控制器
@GetMapping("/unauth")
public String accessDenyPage(){
    return "unauth";
}

3、unauth.html
<body>
    <h1>对不起，您没有访问权限！</h1>
</body>
````
# 三、注解使用
````
使用注解前需要开启注解功能
@EnableGlobalMethodSecurity(securedEnabled = true)

import org.mybatis.spring.annotation.MapperScan;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.security.config.annotation.method.configuration.EnableGlobalMethodSecurity;

@SpringBootApplication
@MapperScan("com.jch.springsecurity.mapper")
//开启注解支持 这个注解也可以放在配置类上面
@EnableGlobalMethodSecurity(securedEnabled = true)
public class SpringSecurityApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringSecurityApplication.class, args);
    }
}
````
* 1、@Secured
````
判断是否具有角色，需要注意：这里匹配字符串需要添加前缀"ROLE_"

1、在controller的方法上加上这个注解
@Secured({"ROLE_sale","ROLE_manager"})
@GetMapping("/update")
public String update() {
    return "hello update";
}

2、我们需要保证在UserDetailsService中含有这个角色才可以
````
* 2、@PreAuthorize
````
适合进入方法前的权限验证，该注解可以将登陆用户的roles/premissions参数传到方法中使用的
位置：方法上

@PreAuthorize("hasAnyAuthority('admin')")
@GetMapping("/update")
public String update() {
    return "hello update";
}
````
* 3、@PostAuthorize
````
这个注解在方法执行之后再进行权限验证，适合验证带有返回值的权限
使用位置：方法上

@PostAuthorize("hasAnyAuthority('aaaa')")//这里的这个权限并不存在 所以会跳转到403页面
@GetMapping("/update")
public String update() {
    System.out.println("update 方法");//该输出会输出到控制台上
    return "hello update";
}
````
* 4、@PostFilter
````
权限验证之后对数据进行过滤，留下用户名是admin1的数据

表达式中的filterObject引用的是方法返回值List中的某一个元素

@GetMapping(value = "/getAll")
@PostAuthorize("hasAnyAuthority('admin')")
@PostFilter("filterObject.username == 'admin1'")
@ResponseBody
public List<Users> getAllUser(){
    ArrayList<Users> list = new ArrayList<>();
    list.add(new Users(1,"admin1","6666"));
    list.add(new Users(2,"admin2","8888"));

    return list;
}
````
* 5、@PreFilter
````
进入控制器前对数据进行过滤
````
# 四、用户注销
````
我们需要在配置类中加入一个退出的地址

//配置退出请求地址
http.logout().logoutUrl("/logout").logoutSuccessUrl("/index").permitAll();
````
# 五、基于数据库的记住我(自动登陆)
* 1、创建表
````
create table persistent_logins (
    username varchar(64) not null, 
    series varchar(64) primary key, 
    token varchar(64) not null, 
    last_used timestamp not null
);
````
* 2、注入数据源，配置操作数据库对象
````
//注入数据源
@Autowired
private DataSource dataSource;

//操作数据库的对象
@Bean
public PersistentTokenRepository persistentTokenRepository(){
    JdbcTokenRepositoryImpl tokenRepositoryImpl = new JdbcTokenRepositoryImpl();
    tokenRepositoryImpl.setDataSource(dataSource);
    //        tokenRepositoryImpl.setCreateTableOnStartup(true);//自动创建表
    return tokenRepositoryImpl;
}

@Override
protected void configure(HttpSecurity http) throws Exception {
    //配置403页面
    http.exceptionHandling().accessDeniedPage("/403_error.html");

    //配置退出请求地址
    http.logout().logoutUrl("/logout").logoutSuccessUrl("/test/hello").permitAll();

    http.formLogin()  //自定义自己编写的登录页面
        .loginPage("/login.html")//登陆页面的地址
        .loginProcessingUrl("/user/login")//登陆页面的请求地址
        .defaultSuccessUrl("/success.html").permitAll()//登陆成功后跳转的页面
        .and()
        .authorizeRequests()//配置权限 哪些地址需要认证，哪些不需要
        .antMatchers("/","/test/hello","/login").permitAll()//设置哪些路径可以直接访问 不需要认证
        .antMatchers("/test/index").hasRole("sale")
        .anyRequest().authenticated()
        
        .and().rememberMe().tokenRepository(persistentTokenRepository())
        .tokenValiditySeconds(60)//以s为单位 设置有效时长
        .userDetailsService(userDetailsService)
        
        .and().csrf().disable();//关闭csrf防护
}
````
* 3、前端的表单信息
````
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>登陆</title>
</head>
<body>
<form action="/user/login" method="POST">
    用户名:<input type="text" name="username"/><br/>
    密码:<input type="password" name="password"/><br/>
    <input type="checkbox" name="remember-me"> <!--自动登陆-->
    <input type="submit" value="登陆"/>
</form>
</body>
</html>
````