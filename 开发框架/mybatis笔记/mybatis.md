# 一、搭建MyBatis
* 1、映射文件的命名规则
  * 表所对应的实体类的类名+Mapper.xml,例如:表t_user,映射的实体类为User,所对应的映射文件为UserMapper.xml
  * MyBatis映射文件用于编写SQL,访问以及操作表中的数据
  * MyBatis映射文件存放的位置是src/main/resources/mappers目录下
  * 映射文件的namespace要和mapper接口的全类名保持一致
  * 映射文件中SQL语句的id要和mapper接口中的方法名一致
* 2、映射文件的编写
  * 查询功能的标签必须设置resultType或resultMap
# 二、MyBatis获取参数值的两种方式
<br>MyBatis获取参数值的两种方式:\${}和#{}
<br>\${}的本质就是字符串拼接,#{}的本质就是占位符赋值
<br>${}使用字符串拼接的方式拼接sql,若为字符串类型或日期类型的字段进行赋值时,需要手动加单引号;
<br>但是#{}使用占位符赋值的方式拼接sql,此时为字符串或日期类型的字段进行赋值时,可以自动添加单引号。

通过\${}和#{}以任意的名称获取参数值,但是需要注意${}的单引号问题。
* 1、mapper接口方法和参数为单个类型:
* 2、mapper接口方法的参数有多个:以arg0,arg1为键,或以param1,param2为键
* 3、mapper接口方法的参数有多个:可以手动将这些参数放在一个map中存储
* 4、mapper接口方法的参数是实体类:
  
<br>通过\${}和#{}以@Param()的value获取参数值,但是需要注意${}的单引号问题。
* 5、使用@Param()注解:
````
User checkLogin(@Param("username") String username, @Param("password") String password);
````

# 三、MyBatis的各种查询功能
* 1、查询一个实体类
````
User getUserById(@Param("id") integer id);

<select id="getUserById" resultType="com.jichenhao.mybatis.pojo.User">
  select * from t_user where id = #{id}
</select> 
````
* 2、查询一个list集合
````
List<User> getAllUser();

<select id="getAllUser" resultType="com.jichenhao.mybatis.pojo.User">
  select * from t_user
</select> 
````
* 3、查询一个用户结果转为集合
````
Map<String,Object> getUserByIdToMap(@Param("id") Integer id);

<select id="getUserByIdToMap" resultType="map">
  select * from t_user where id = #{id}
</select>
````
* 4、查询多个用户结果转为集合
````
@MapKey("id")
Map<String,Object> getAllUserToMap();

<select id="getAllUserToMap" resultType="map">
  select * from t_user
</select>
````
# 四、特殊SQL的执行
* 1、模糊SQL
````
List<User> getUserByLike(@Param("username") String username);

<select id="getUserByLike" resultType="com.jichenhao.mybatis.pojo.User">
  select * from t_user where username like "%"#{username}"%"
  或者 select * from t_user where username like '%${username}%'
  或者 select * from t_user where username like concat('%',#{username},'%')
</select>
````
* 2、批量删除
````
int deleteMore(@Param("ids") String ids);

<delete id = "deleteMore">
  delete from t_user where id in ($(ids))
</delete>
````
* 3、动态设置表名
````
List<User> getAllUser(@Param("tableName") String tableName);

<select id="getAllUser" resultType="com.jichenhao.mybatis.pojo.User">
  select * from ${tableName}
</select>

````
* 4、添加功能获取自增的主键
````
void insertUser(User user);
//userGeneratedKeys="true" 设置主键自增,keyProperty="id" 将表的主键赋值到User的id
<select id="insertUser" resultType="com.jichenhao.mybatis.pojo.User" userGeneratedKeys="true" keyProperty="id">
  insert into t_user values(null,#{username},#{password},#{age},#{sex},#{email})
</select>
````
# 五、自定义映射
* 1、解决字段名和属性名不一致的情况
  * a>为字段起别名,保持和属性名的一致
  * b>设置全局配置,将_自动映射为驼峰 
  ````
  设置MyBatis的全局配置
  <settings>
    //将下划线映射为驼峰
    <setting name="mapUnderscoreToCamelCass" value="true">
  </settings>
  ````
  * c>resultMap设置自定义的映射关系
  ````
  //property:设置映射关系中实体类的属性名。column=:设置映射关系中的字段名,必须是sql语句查询出的字段名。
  <resultMap id="empResultMap" type="Emp">
    <id property="eid" column="eid"></id>
    <result property="empName" column="emp_name"></result>
    <result property="age" column="age"></result>
    <result property="sex" column="sex"></result>
    <result property="email" column="email"></result>
  </resultMap>

  <select id="getAllEmp" resultMap="empResultMap">
    select * from t_emp
  </select>
  ````
* 2、处理多对一映射关系(查找员工所在部门)
  * a>级联属性赋值
  * b>association
  ````
  <resultMap id="empAndDeptResultMap" type="Emp">
    <id property="eid" column="eid"></id>
    <result property="empName" column="emp_name"></result>
    <result property="age" column="age"></result>
    <result property="sex" column="sex"></result>
    <result property="email" column="email"></result>
    //property:需要处理多对一映射关系的属性名。javaType:该属性的类型。
    <association property="dept" javaType="Dept">
      <id property="did" column="did"></id>
      <result property="deptName" column="dept_name"></result>
    </association>
  </resultMap>

  <select id="getEmpAndDept" resultMap="empAndDeptResultMap">
    select * from t_emp left join t_dept on t_emp.did = t_dept.did where t_emp.eid = #{eid}
  </select>
  ````
  * c>分步查询
  <br>分布查询可以实现延迟加载，默认不开启，必须在核心配置文件中设置全局配置信息：
  <br>lazyLoadingEnabled：延迟加载的全局开关。当开启时，所有关联对象都会延迟加载
    ````
    <settings>
          <!--将_自动映射为驼峰，emp_name:empName-->
          <setting name="mapUnderscoreToCamelCase" value="true"/>
          <!--开启延迟加载-->
          <setting name="lazyLoadingEnabled" value="true"/>
    </settings>
    ````
    <br>aggressiveLazyLoading：当开启时，任何方法的调用都会加载该对象的所有属性 否则，每个属性会按需加载，此时就可以实现按需加载，获取的数据是什么，就只会执行相应的sql。
    <br>此时可通过association中的fetchType属性设置当前的分步查询是否使用延迟加载，fetchType="lazy(延迟加载)|eager(立即加载)"
    ````
    //第一步 EmpMapper.xml 先查员工ID(eid)
    <resultMap id="empAndDeptByStepResultMap" type="Emp">
          <id property="eid" column="eid"></id>
          <result property="empName" column="emp_name"></result>
          <result property="age" column="age"></result>
          <result property="sex" column="sex"></result>
          <result property="email" column="email"></result>
          <!--
              select:设置分步查询的sql的唯一标识（namespace.SQLId或mapper接口的全类名.方法名）
              column:设置分步查询的条件
              fetchType:当开启了全局的延迟加载之后，可通过此属性手动控制延迟加载的效果
              fetchType="lazy|eager":lazy表示延迟加载，eager表示立即加载
          -->
          <association property="dept"
                      select="com.jch.mybatis.mapper.DeptMapper.getEmpAndDeptByStepTwo"
                      column="did"
                      fetchType="eager"></association>
    </resultMap>
    
    <!--Emp getEmpAndDeptByStepOne(@Param("eid") Integer eid);-->
      <select id="getEmpAndDeptByStepOne" resultMap="empAndDeptByStepResultMap">
          select * from t_emp where eid = #{eid}
      </select>
    //第二步 DeptMapper.xml 通过did查询员工所对应的部门
    <select id="getEmpAndDeptByStepTwo" resultType="Dept">
          select * from t_dept where did = #{did}
    </select>
    ````
* 3、处理一对多映射关系(查找部门有哪些员工)
  * a>collection
  ````
  <resultMap id="deptAndEmpResultMap" type="Dept">
      <id property="did" column="did"></id>
      <result property="deptName" column="dept_name"></result>
      <!--
          collection：处理一对多的映射关系
          ofType：表示该属性所对应的集合中存储数据的类型
          emps:部门实体类中定义的员工列表对象
      -->
      <collection property="emps" ofType="Emp">
          <id property="eid" column="eid"></id>
          <result property="empName" column="emp_name"></result>
          <result property="age" column="age"></result>
          <result property="sex" column="sex"></result>
          <result property="email" column="email"></result>
      </collection>
  </resultMap>
  <select id="getDeptAndEmp" resultMap="deptAndEmpResultMap">
    select * from t_dept left join t_emp on t_dept.did = t_emp.did where t_dept.did = #{did}
  </select>
  ````
  * b>分步查询
  ````
  //分步查询第一步：查询部门信息
    <resultMap id="deptAndEmpByStepResultMap" type="Dept">
        <id property="did" column="did"></id>
        <result property="deptName" column="dept_name"></result>
        <collection property="emps"
                    select="com.atguigu.mybatis.mapper.EmpMapper.getDeptAndEmpByStepTwo"
                    column="did" fetchType="eager"></collection>
    </resultMap>
    <select id="getDeptAndEmpByStepOne" resultMap="deptAndEmpByStepResultMap">
        select * from t_dept where did = #{did}
    </select>
  //分步查询第二步：通过did查询员工所对应的部门
    <select id="getDeptAndEmpByStepTwo" resultType="Emp">
        select * from t_emp where did = #{did}
    </select>

  ````
# 六、动态sql
* 1、if
````
List<Emp> getEmpByCondition(Emp emp);

<select id="getEmpByCondition" resultType="Emp">
  select * from t_user where 1 = 1
  <if test="empName != null and empName != ''">emp_name = #{empName}</if>
  <if test="age != null and age != ''">age = #{age}</if>
  <if test="sex != null and sex != ''">sex = #{sex}</if>
  <if test="email != null and email != ''">email = #{email}</if>
</select>
````
* 2、where(可以去掉内容前多余的and和or)
````
<select id="getEmpByCondition" resultType="Emp">
  select * from t_user
  <where>
    <if test="empName != null and empName != ''">emp_name = #{empName}</if>
    <if test="age != null and age != ''"> and age = #{age}</if>
    <if test="sex != null and sex != ''"> and sex = #{sex}</if>
    <if test="email != null and email != ''"> and email = #{email}</if>
  </where>
</select>
````
* 3、trim
  * prefix/suffix:将trim标签中内容前面或者后面添加指定内容
  * prefixOverrides/suffixOverrides:将trim标签中内容前面或者后面去掉指定内容
  ````
  <select id="getEmpByCondition" resultType="Emp">
    select * from t_user
    <trim prefix="where" suffixOverrides="and|or">
      <if test="empName != null and empName != ''">emp_name = #{empName} and</if>
      <if test="age != null and age != ''">age = #{age} or</if>
      <if test="sex != null and sex != ''">sex = #{sex} and</if>
      <if test="email != null and email != ''">email = #{email}</if>
    </trim>
  </select>
  ````
* 4、choose,when,otherwise(相当于 if...else if...else)
````
List<Emp> getEmpByChoose(Emp emp);

<select id="getEmpByChoose" resultType="Emp">
  select * from t_user
  <where>
    <choose>
      <when test="empName != null and empName != ''">emp_name = #{empName}</when>
      <when test="age != null and age != ''">age = #{age}</when>
      <when test="sex != null and sex != ''">sex = #{sex}</when>
      <when test="email != null and email != ''">email = #{email}</when>
      <otherwise>did=1</otherwise>
    </choose>
  </where>
</select>
````
* 5、foreach
````
批量删除
int deleteMoreByArray(@Param("eids")Integer[] eids);

<delete id="deleteMoreByArray">
  delete from t_emp where eid in 
  <foreach collection="eids" item="eid" separator="," open="(" close=")">
    #{eid}
  </foreach>
</delete>
或者
<delete id="deleteMoreByArray">
  delete from t_emp where
  <foreach collection="eids" item="eid" separator="or">
    eid = #{eid}
  </foreach>
</delete>
````
````
批量添加
int insertMoreByList(@Param("emps")List<Emp> emps);

<insert id="insertMoreByList">
  insert into t_emp values
  <foreach collection="emps" item="emp" separator=",">
    (null,#{emp.empName},#{emp.age},#{emp.sex},#{emp.email},null)
  </foreach>
</insert>
````
* 6、sql
````
设置SQL片段:<sql id="empColumns">eid,emp_name,age,sex,email</sql>
引用SQL片段:<include refid="empColumns"></include>
````
# 七、MyBatis的缓存
* 1、MyBatis的一级缓存
<br>一级缓存是SqlSession级别的,通过同一个SqlSession查询的数据会被缓存,下次查询相同的数据,就会从缓存中直接获取,不会从数据库重新访问
<br>使一级缓存失效的四种情况:
<br>不同的SqlSession对应不同的一级缓存
<br>同一个SqlSession但是查询条件不同
<br>同一个SqlSession两次查询期间执行了任何一次的增删改操作
<br>同一个SqlSession两次查询期间手动清空了缓存

* 2、MyBatis的二级缓存
<br>二级缓存是SqlSessionFactory级别，通过同一个SqlSessionFactory创建的SqlSession查询的结果会被缓存；此后若再次执行相同的查询语句，结果就会从缓存中获取。
<br>a>在核心配置文件中，设置全局配置属性cacheEnabled="true"，默认为true，不需要设置
<br>b>在映射文件中设置标签\<cache />
<br>c>二级缓存必须在SqlSession关闭或提交之后有效sqlsession.commit() sqlsession.close()操作之前在一级缓存
<br>d>查询的数据所转换的实体类类型必须实现序列化的接口。实体类 implements Serializable
<br>使二级缓存失效的情况：两次查询之间执行了任意的增删改，会使一级和二级缓存同时失效

* 3、MyBatis缓存查询的顺序
<br>1.先查询二级缓存，因为二级缓存中可能会有其他程序已经查出来的数据，可以拿来直接使用。
<br>2.如果二级缓存没有命中，再查询一级缓存
<br>3.如果一级缓存也没有命中，则查询数据库
<br>SqlSession关闭之后，一级缓存中的数据会写入二级缓存

* 4、MyBatis的逆向工程
<br>a>引入依赖
````
<!-- 依赖MyBatis核心包 -->
<dependencies>
<dependency>
<groupId>org.mybatis</groupId>
<artifactId>mybatis</artifactId>
<version>3.5.7</version>
</dependency>
</dependencies>
<!-- 控制Maven在构建过程中相关配置 -->
<build>
<!-- 构建过程中用到的插件 -->
<plugins>
<!-- 具体插件，逆向工程的操作是以构建过程中插件形式出现的 -->
<plugin>
<groupId>org.mybatis.generator</groupId>
<artifactId>mybatis-generator-maven-plugin</artifactId>
<version>1.3.0</version>
<!-- 插件的依赖 -->
<dependencies>
<!-- 逆向工程的核心依赖 -->
<dependency>
<groupId>org.mybatis.generator</groupId>
<artifactId>mybatis-generator-core</artifactId>
<version>1.3.2</version>
</dependency>
<!-- 数据库连接池 -->
<dependency>
<groupId>com.mchange</groupId>
<artifactId>c3p0</artifactId>
<version>0.9.2</version>
</dependency>
<!-- MySQL驱动 -->
<dependency>
<groupId>mysql</groupId>
<artifactId>mysql-connector-java</artifactId>
<version>5.1.8</version>
</dependency>
</dependencies>
</plugin>
</plugins>
</build>
````
<br>b>创建逆向工程的配置文件,文件名必须是：generatorConfig.xml
<br>targetRuntime: 执行生成的逆向工程的版本
<br>MyBatis3Simple: 生成基本的CRUD（清新简洁版）MyBatis3: 生成带条件的CRUD（奢华尊享版）
````
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE generatorConfiguration
        PUBLIC "-//mybatis.org//DTD MyBatis Generator Configuration 1.0//EN"
        "http://mybatis.org/dtd/mybatis-generator-config_1_0.dtd">
<generatorConfiguration>
    <!--
            targetRuntime: 执行生成的逆向工程的版本
                    MyBatis3Simple: 生成基本的CRUD（清新简洁版）
                    MyBatis3: 生成带条件的CRUD（奢华尊享版）
     -->
    <context id="DB2Tables" targetRuntime="MyBatis3">
        <!-- 数据库的连接信息 -->
        <jdbcConnection driverClass="com.mysql.jdbc.Driver"
                        connectionURL="jdbc:mysql://localhost:3306/mybatis"
                        userId="root"
                        password="123456">
        </jdbcConnection>
        <!-- javaBean的生成策略-->
        <javaModelGenerator targetPackage="com.atguigu.mybatis.pojo" targetProject=".\src\main\java">
            <property name="enableSubPackages" value="true" /><!--是否能使用子包 true代表每个点为一层包，false则代表为一个包名-->
            <property name="trimStrings" value="true" /><!-- 自动去掉字符串空格-->
        </javaModelGenerator>
        <!-- SQL映射文件的生成策略 -->
        <sqlMapGenerator targetPackage="com.atguigu.mybatis.mapper"  targetProject=".\src\main\resources">
            <property name="enableSubPackages" value="true" />
        </sqlMapGenerator>
        <!-- Mapper接口的生成策略 -->
        <javaClientGenerator type="XMLMAPPER" targetPackage="com.atguigu.mybatis.mapper"  targetProject=".\src\main\java">
            <property name="enableSubPackages" value="true" />
        </javaClientGenerator>
        <!-- 逆向分析的表 -->
        <!-- tableName设置为*号，可以对应所有表，此时不写domainObjectName -->
        <!-- domainObjectName属性指定生成出来的实体类的类名 -->
        <table tableName="t_emp" domainObjectName="Emp"/>
        <table tableName="t_dept" domainObjectName="Dept"/>
    </context>
</generatorConfiguration>
````
<br>c>执行MGB插件的generate目标

![本地路径](执行MGB插件的generate目标.PNG)
# 八、分页插件的使用
* 1、添加依赖
````
<dependency>
<groupId>com.github.pagehelper</groupId>
<artifactId>pagehelper</artifactId>
<version>5.2.0</version>
</dependency>
````
* 2、在MyBatis的核心配置文件中配置插件
````
<plugins>
<!--设置分页插件-->
<plugin interceptor="com.github.pagehelper.PageInterceptor"></plugin>
</plugins>
````
* 3、分页插件的使用
````
a>在查询功能之前使用PageHelper.startPage(int pageNum, int pageSize)开启分页功能
pageNum：当前页的页码
pageSize：每页显示的条数
b>在查询获取list集合之后，使用PageInfo<T> pageInfo = new PageInfo<>(List<T> list, int
navigatePages)获取分页相关数据
list：分页之后的数据
navigatePages：导航分页的页码数
c>分页相关数据
PageInfo{
pageNum=8, pageSize=4, size=2, startRow=29, endRow=30, total=30, pages=8,
list=Page{count=true, pageNum=8, pageSize=4, startRow=28, endRow=32, total=30,
pages=8, reasonable=false, pageSizeZero=false},
prePage=7, nextPage=0, isFirstPage=false, isLastPage=true, hasPreviousPage=true,
hasNextPage=false, navigatePages=5, navigateFirstPage4, navigateLastPage8,
navigatepageNums=[4, 5, 6, 7, 8]
}
````
* 4、常用数据
````
pageNum：当前页的页码
pageSize：每页显示的条数
size：当前页显示的真实条数
total：总记录数
pages：总页数
prePage：上一页的页码
nextPage：下一页的页码
isFirstPage/isLastPage：是否为第一页/最后一页
hasPreviousPage/hasNextPage：是否存在上一页/下一页
navigatePages：导航分页的页码数
navigatepageNums：导航分页的页码，[1,2,3,4,5]
````