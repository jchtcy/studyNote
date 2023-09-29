# 一、MyBatisplus实现
* 1、实体类User
安装lombok插件后，Data注解实现实体类User的无参构造、get/set、equals、hashcode、toString方法,但不能实现有参构造。
````
@Data
public class User {
    private Long id;
    private String name;
    private Integer age;
    private String email;
 
}
````
* 2、mapper层的UserMapper接口
直接继承由mybatisplus提供的BaseMapper（该类为我们提供了一系列增删查改的方法）
````
public interface UserMaper extends BaseMapper<User> {
}
````
* 3、配置application.yml
````
spring:
    # 配置数据源信息
    datasource:
        # 配置数据源类型
        type: com.zaxxer.hikari.HikariDataSource
        # 配置连接数据库信息
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://localhost:3306/mybatis_plus?characterEncoding=utf-
        8&useSSL=false
        username: root
        password: 123456
    # 配置MyBatis日志
    mybatis-plus:
        configuration:
            log-impl: org.apache.ibatis.logging.stdout.StdOutImpl

````
# 二、Sevice CRUD接口
````
通用Service CRUD封装IService接口，进一步封装CRUD采用get查询单行remove删除,list查询集合page分页前缀命名方式区分Mapper层避免混淆,泛型τ为任意实体对象。
建议如果存在自定义通用Service方法的可能，请创建自己的IBaseService 继承Mybatis-Plus 提供的基类。
对象Wrapper 为条件构造器。
````
* 1、创建UserService接口继承MyBtis-Plus提供的IService
````
public interface UserService extends IService<User> {
}
````
* 2、创建UserServiceImpl实现类，继承MyBtis-Plus提供的ServiceImpl实现类，实现UserService接口
````
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {
}
````
# 三、常见注解
* 1、@TableName
在实体类上添加@tableName注解，标识实体类对应的表。因为MyBatis-Plus在确定操作的表时，由BaseMapper的泛型决定，即实体类型决定，且默认操作的表名和实体类型的类名一致，如果不一致就会发生错误，这时就需要用@tableName注解指定该实体类对应的表了。
````
当实体类所对应的表都有固定的前缀，例如t_时，可以使用MyBatis-Plus提供的全局配置，为实体类所对应的表名设置默认的前缀

# 设置MyBatis-Plus的全局配置
global-config:
    db-config:
        #设置实体类对应的表的统一前缀
        table-prefix:t_
````
* 2、@TableId
将属性所对应的字段指定为主键
````
@TableId
private Long uid;
````
@TableId注解的value属性:用于指定主键的字段（当实体类中表示主键的字段为id时,但数据库中的主键为uid,可以通过@TableId注解的value属性指定主键字段名为数据库中的主键字段名）
````
@TableId(value = "uid")
private Long id;
````
@TableId注解的type属性:设置主键的生成策略 
````
配置全局主键策略
global-config:
    db-config:
        # 配置MyBatis-Plus操作表的默认前缀
        table-prefix: t_
        # 配置MyBatis-Plus的主键策略
        id-type: auto
````
* 3、@TableField
````
a>若实体类中的属性使用的是驼峰命名风格，而表中的字段使用的是下划线命名风格
例如实体类属性userName，表中字段user_name
此时MyBatis-Plus会自动将下划线命名风格转化为驼峰命名风格
相当于在MyBatis中配置
b>若实体类中的属性和表中的字段不满足a中条件
例如实体类属性name，表中字段username
此时需要在实体类属性上使用@TableField("username")设置属性所对应的字段名
@Data
public class User {
    private Long id;
    @TableField("username")
    private String name;
    private Integer age;
    private String email;
 
}
````
* 4、@TableLogic
````
a>逻辑删除
物理删除：真实删除，将对应数据从数据库中删除，之后查询不到此条被删除的数据
逻辑删除：假删除，将对应数据中代表是否被删除字段的状态修改为“被删除状态”，之后在数据库
中仍旧能看到此条数据记录
使用场景：可以进行数据恢复
b>实现逻辑删除
step1：数据库中创建逻辑删除状态列，设置默认值为0
step2：实体类中添加逻辑删除属性
@Data
public class User {
    private Long id;
    @TableField("username")
    private String name;
    private Integer age;
    private String email;
    @TableLogic
    private Integer isDeleted;
}

````
# 四、条件构造器和常用接口
* 1、wapper介绍
````
Wrapper ： 条件构造抽象类，最顶端父类
    AbstractWrapper ： 用于查询条件封装，生成 sql 的 where 条件
        QueryWrapper ： 查询条件封装
        UpdateWrapper ： Update 条件封装
        AbstractLambdaWrapper ： 使用Lambda 语法
            LambdaQueryWrapper ：用于Lambda语法使用的查询Wrapper
            LambdaUpdateWrapper ： Lambda 更新封装Wrapper
````
* 2、QueryWrapper
````
a>例1：组装查询条件
@Test
public void test01(){
    //查询用户名包含a，年龄在20到30之间，并且邮箱不为null的用户信息
    //SELECT id,username AS name,age,email,is_deleted FROM t_user WHERE is_deleted=0 AND (username LIKE ? AND age BETWEEN ? AND ? AND email IS NOT NULL)
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    queryWrapper.like("username", "a")
        .between("age", 20, 30)
        .isNotNull("email");
    List<User> list = userMapper.selectList(queryWrapper);
    list.forEach(System.out::println);
}
````
````
b>例2：组装排序条件
@Test
public void test02(){
    //按年龄降序查询用户，如果年龄相同则按id升序排列
    //SELECT id,username AS name,age,email,is_deleted FROM t_user WHERE is_deleted=0 ORDER BY age DESC,id ASC
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    queryWrapper
        .orderByDesc("age")
        .orderByAsc("id");
    List<User> users = userMapper.selectList(queryWrapper);
    users.forEach(System.out::println);
}
````
````
c>例3：组装删除条件
@Test
public void test03(){
    //删除email为空的用户
    //DELETE FROM t_user WHERE (email IS NULL)
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    queryWrapper.isNull("email");
    //条件构造器也可以构建删除语句的条件
    int result = userMapper.delete(queryWrapper);
    System.out.println("受影响的行数：" + result);
}
````
````
d>例4：条件的优先级
@Test
public void test04() {
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    //将（年龄大于20并且用户名中包含有a）或邮箱为null的用户信息修改
    //UPDATE t_user SET age=?, email=? WHERE (username LIKE ? AND age > ? OR email IS NULL)
    queryWrapper
        .like("username", "a")
        .gt("age", 20)
        .or()
        .isNull("email");
    User user = new User();
    user.setAge(18);
    user.setEmail("user@atguigu.com");
    int result = userMapper.update(user, queryWrapper);
    System.out.println("受影响的行数：" + result);
}

@Test
public void test04() {
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    //将用户名中包含有a并且（年龄大于20或邮箱为null）的用户信息修改
    //UPDATE t_user SET age=?, email=? WHERE (username LIKE ? AND (age > ? OR email IS NULL))
    //lambda表达式内的逻辑优先运算
    queryWrapper
        .like("username", "a")
        .and(i -> i.gt("age", 20).or().isNull("email"));
    User user = new User();
    user.setAge(18);
    user.setEmail("user@atguigu.com");
    int result = userMapper.update(user, queryWrapper);
    System.out.println("受影响的行数：" + result);
}
````
````
e>例5：组装select子句
@Test
public void test05() {
    //查询用户信息的username和age字段
    //SELECT username,age FROM t_user
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    queryWrapper.select("username", "age");
    //selectMaps()返回Map集合列表，通常配合select()使用，避免User对象中没有被查询到的列值为null
    List<Map<String, Object>> maps = userMapper.selectMaps(queryWrapper);
    maps.forEach(System.out::println);
}
````
````
f>例6：实现子查询
@Test
public void test06() {
    //查询id小于等于3的用户信息
    //SELECT id,username AS name,age,email,is_deleted FROM t_user WHERE (id IN (select id from t_user where id <= 3))
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    queryWrapper.inSql("id", "select id from t_user where id <= 3");
    List<User> list = userMapper.selectList(queryWrapper);
    list.forEach(System.out::println);
}
````
* 3、UpdateWrapper
````
@Test
public void test07() {
    //将（年龄大于20或邮箱为null）并且用户名中包含有a的用户信息修改
    //组装set子句以及修改条件
    UpdateWrapper<User> updateWrapper = new UpdateWrapper<>();
    //lambda表达式内的逻辑优先运算
    updateWrapper
        .like("username", "a")
        .and(i -> i.gt("age", 20).or().isNull("email"))
        .set("age", 18)
        .set("email", "user@atguigu.com");
    //UPDATE t_user SET age=?,email=? WHERE (username LIKE ? AND (age > ? OR email IS NULL))
    int result = userMapper.update(null, updateWrapper);
    System.out.println(result);
}
````
* 4、condition
````
在真正开发的过程中，组装条件是常见的功能，而这些条件数据来源于用户输入，是可选的，因
此我们在组装这些条件时，必须先判断用户是否选择了这些条件，若选择则需要组装该条件，若
没有选择则一定不能组装，以免影响SQL执行的结果
@Test
public void test08UseCondition() {
    //定义查询条件，有可能为null（用户未输入或未选择）
    String username = null;
    Integer ageBegin = 10;
    Integer ageEnd = 24;
    QueryWrapper<User> queryWrapper = new QueryWrapper<>();
    //StringUtils.isNotBlank()判断某字符串是否不为空且长度不为0且不由空白符(whitespace)构成
    queryWrapper
        .like(StringUtils.isNotBlank(username), "username", "a")
        .ge(ageBegin != null, "age", ageBegin)
        .le(ageEnd != null, "age", ageEnd);
    //SELECT id,username AS name,age,email,is_deleted FROM t_user WHERE (age >= ? AND age <= ?)
    List<User> users = userMapper.selectList(queryWrapper);
    users.forEach(System.out::println);
}
````
* 5、LambdaQueryWrapper
````
@Test
public void test09() {
    //定义查询条件，有可能为null（用户未输入）
    String username = "a";
    Integer ageBegin = 10;
    Integer ageEnd = 24;
    LambdaQueryWrapper<User> queryWrapper = new LambdaQueryWrapper<>();
    //避免使用字符串表示字段，防止运行时错误
    queryWrapper
        .like(StringUtils.isNotBlank(username), User::getName, username)
        .ge(ageBegin != null, User::getAge, ageBegin)
        .le(ageEnd != null, User::getAge, ageEnd);
    List<User> users = userMapper.selectList(queryWrapper);
    users.forEach(System.out::println);
}
````
* 6、LambdaUpdateWrapper
````
@Test
public void test10() {
//组装set子句
LambdaUpdateWrapper<User> updateWrapper = new LambdaUpdateWrapper<>();
    updateWrapper
        .like(User::getName, "a")
        .and(i -> i.lt(User::getAge, 24).or().isNull(User::getEmail)) //lambda表达式内的逻辑优先运算
        .set(User::getAge, 18)
        .set(User::getEmail, "user@atguigu.com");
    User user = new User();
    int result = userMapper.update(user, updateWrapper);
    System.out.println("受影响的行数：" + result);
}
````
# 六、插件
* 1、分页插件
````
MyBatis Plus自带分页插件，只要简单的配置即可实现分页功能

a>添加配置类
@Configuration
@MapperScan("com.atguigu.mybatisplus.mapper") //可以将主类中的注解移到此处
public class MybatisPlusConfig {
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
        return interceptor;
    }
}

b>测试
@Test
public void testPage(){
    //设置分页参数
    Page<User> page = new Page<>(1, 5);
    userMapper.selectPage(page, null);
    //获取分页数据
    List<User> list = page.getRecords();
    list.forEach(System.out::println);
    System.out.println("当前页："+page.getCurrent());
    System.out.println("每页显示的条数："+page.getSize());
    System.out.println("总记录数："+page.getTotal());
    System.out.println("总页数："+page.getPages());
    System.out.println("是否有上一页："+page.hasPrevious());
    System.out.println("是否有下一页："+page.hasNext());
}

结果:
User(id=1, name=Jone, age=18, email=test1@baomidou.com, isDeleted=0) 
User(id=2,name=Jack, age=20, email=test2@baomidou.com, isDeleted=0) 
User(id=3, name=Tom,age=28, email=test3@baomidou.com, isDeleted=0) 
User(id=4, name=Sandy, age=21,email=test4@baomidou.com, isDeleted=0) 
User(id=5, name=Billie, age=24, email=test5@baomidou.com, isDeleted=0) 
当前页：1 
每页显示的条数：5 
总记录数：17 
总页数：4 
是否有上一页：false 
是否有下一页：true
````
* 2、xml自定义分页
````
a>UserMapper中定义接口方法
/**
* 根据年龄查询用户列表，分页显示
* @param page 分页对象,xml中可以从里面进行取值,传递参数 Page 即自动分页,必须放在第一位
* @param age 年龄
* @return
*/
    IPage<User> selectPageVo(@Param("page") Page<User> page, @Param("age") Integer age);

b>UserMapper.xml中编写SQL

<sql id="BaseColumns">id,username,age,email</sql>

<select id="selectPageVo" resultType="User">
    SELECT <include refid="BaseColumns"></include> FROM t_user WHERE age > #{age}
</select>

c>测试
@Test
public void testSelectPageVo(){
    //设置分页参数
    Page<User> page = new Page<>(1, 5);
    userMapper.selectPageVo(page, 20);
    //获取分页数据
    List<User> list = page.getRecords();
    list.forEach(System.out::println);
    System.out.println("当前页："+page.getCurrent());
    System.out.println("每页显示的条数："+page.getSize());
    System.out.println("总记录数："+page.getTotal());
    System.out.println("总页数："+page.getPages());
    System.out.println("是否有上一页："+page.hasPrevious());
    System.out.println("是否有下一页："+page.hasNext());
}

结果:
User(id=3, name=Tom, age=28, email=test3@baomidou.com, isDeleted=null) 
User(id=4, name=Sandy, age=21, email=test4@baomidou.com, isDeleted=null) 
User(id=5, name=Billie, age=24, email=test5@baomidou.com, isDeleted=null) 
User(id=8, name=ybc1, age=21, email=null, isDeleted=null) 
User(id=9, name=ybc2, age=22, email=null, isDeleted=null) 
当前页：1 
每页显示的条数：5 
总记录数：12 
总页数：3 
是否有上一页：false 
是否有下一页：true
````
* 3、乐观锁
````
a>修改实体类
package com.atguigu.mybatisplus.entity;
import com.baomidou.mybatisplus.annotation.Version;
import lombok.Data;
@Data
public class Product {
    private Long id;
    private String name;
    private Integer price;
    @Version
    private Integer version;
}

b>添加乐观锁插件配置
@Bean
public MybatisPlusInterceptor mybatisPlusInterceptor(){
    MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
    //添加分页插件
    interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
    //添加乐观锁插件
    interceptor.addInnerInterceptor(new OptimisticLockerInnerInterceptor());
    return interceptor;
}

c>优化流程
@Test
public void testConcurrentVersionUpdate() {
    //小李取数据
    Product p1 = productMapper.selectById(1L);
    //小王取数据
    Product p2 = productMapper.selectById(1L);
    //小李修改 + 50
    p1.setPrice(p1.getPrice() + 50);
    int result1 = productMapper.updateById(p1);
    System.out.println("小李修改的结果：" + result1);
    //小王修改 - 30
    p2.setPrice(p2.getPrice() - 30);
    int result2 = productMapper.updateById(p2);
    System.out.println("小王修改的结果：" + result2);
    if(result2 == 0){
        //失败重试，重新获取version并更新
        p2 = productMapper.selectById(1L);
        p2.setPrice(p2.getPrice() - 30);
        result2 = productMapper.updateById(p2);
    }
    System.out.println("小王修改重试的结果：" + result2);
    //老板看价格
    Product p3 = productMapper.selectById(1L);
    System.out.println("老板看价格：" + p3.getPrice());
}
````
# 七、通用枚举
* 1、数据库表添加字段sex
* 2、创建通用枚举类型
````
package com.atguigu.mp.enums;
import com.baomidou.mybatisplus.annotation.EnumValue;
import lombok.Getter;
@Getter
public enum SexEnum {
    MALE(1, "男"),
    FEMALE(2, "女");
    @EnumValue
    private Integer sex;
    private String sexName;
    SexEnum(Integer sex, String sexName) {
        this.sex = sex;
        this.sexName = sexName;
    }
}
````
* 3、配置扫描通用枚举
````
mybatis-plus:
    configuration:
        # 配置MyBatis日志
        log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
    global-config:
        db-config:
            # 配置MyBatis-Plus操作表的默认前缀
            table-prefix: t_
            # 配置MyBatis-Plus的主键策略
            id-type: auto
    # 配置扫描通用枚举
    type-enums-package: com.atguigu.mybatisplus.enums
````
* 4、测试
````
@Test
public void testSexEnum(){
    User user = new User();
    user.setName("Enum");
    user.setAge(20);
    //设置性别信息为枚举项，会将@EnumValue注解所标识的属性值存储到数据库
    user.setSex(SexEnum.MALE);
    //INSERT INTO t_user ( username, age, sex ) VALUES ( ?, ?, ? )
    //Parameters: Enum(String), 20(Integer), 1(Integer)
    userMapper.insert(user);
}
````
# 八、代码生成器
* 1、引入依赖
````
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-generator</artifactId>
    <version>3.5.1</version>
</dependency>
<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
    <version>2.3.31</version>
</dependency>
````
* 2、快速生成
````
public class FastAutoGeneratorTest {
    public static void main(String[] args) {
        FastAutoGenerator.create("jdbc:mysql://127.0.0.1:3306/mybatis_plus?characterEncoding=utf-8&userSSL=false", "root", "123456")
            .globalConfig(builder -> {
                builder.author("atguigu") // 设置作者
                //.enableSwagger() // 开启 swagger 模式
                .fileOverride() // 覆盖已生成文件
                .outputDir("D://mybatis_plus"); // 指定输出目录
            })
            .packageConfig(builder -> {
                builder.parent("com.atguigu") // 设置父包名
                    .moduleName("mybatisplus") // 设置父包模块名
                    .pathInfo(Collections.singletonMap(OutputFile.mapperXml, "D://mybatis_plus"));// 设置mapperXml生成路径
            })
            .strategyConfig(builder -> {
                builder.addInclude("t_user") // 设置需要生成的表名
                .addTablePrefix("t_", "c_"); // 设置过滤表前缀
            })
            .templateEngine(new FreemarkerTemplateEngine()) // 使用Freemarker引擎模板，默认的是Velocity引擎模板
            .execute();
    }
}
````
# 九、多数据源
* 1、创建数据库及表
````
创建数据库mybatis_plus_1和表product
CREATE DATABASE `mybatis_plus_1` /*!40100 DEFAULT CHARACTER SET utf8mb4 */;
use `mybatis_plus_1`;
CREATE TABLE product
(
    id BIGINT(20) NOT NULL COMMENT '主键ID',
    name VARCHAR(30) NULL DEFAULT NULL COMMENT '商品名称',
    price INT(11) DEFAULT 0 COMMENT '价格',
    version INT(11) DEFAULT 0 COMMENT '乐观锁版本号',
    PRIMARY KEY (id)
);
INSERT INTO product (id, NAME, price) VALUES (1, '外星人笔记本', 100);
````
* 2、引入依赖
````
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>dynamic-datasource-spring-boot-starter</artifactId>
    <version>3.5.0</version>
</dependency>
````
* 3、配置多数据源
````
spring:
    # 配置数据源信息
    datasource:
        dynamic:
            # 设置默认的数据源或者数据源组,默认值即为master
            primary: master
            # 严格匹配数据源,默认false.true未匹配到指定数据源时抛异常,false使用默认数据源
            strict: false
            datasource:
                master:
                    url: jdbc:mysql://localhost:3306/mybatis_plus?characterEncoding=utf-8&useSSL=false
                    driver-class-name: com.mysql.cj.jdbc.Driver
                    username: root
                    password: 123456
                slave_1:
                    url: jdbc:mysql://localhost:3306/mybatis_plus_1?characterEncoding=utf-8&useSSL=false
                    driver-class-name: com.mysql.cj.jdbc.Driver
                    username: root
                    password: 123456
````
* 4、创建用户service
````
public interface UserService extends IService<User> {
}

@DS("master") //指定所操作的数据源
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {
}
````
* 5、创建商品service
````
public interface ProductService extends IService<Product> {
}

@DS("slave_1")
@Service
public class ProductServiceImpl extends ServiceImpl<ProductMapper, Product> implements ProductService {
}
````
* 6、测试
````
@Autowired
private UserService userService;
@Autowired
private ProductService productService;
@Test
public void testDynamicDataSource(){
    System.out.println(userService.getById(1L));
    System.out.println(productService.getById(1L));
}
````