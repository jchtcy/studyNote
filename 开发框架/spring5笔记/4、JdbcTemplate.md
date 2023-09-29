# JdbcTemplate
* 1、引入依赖
````
mysql-connector-java-5.1.7-bin
spring-jdbc-5.2.6.RELEASE
spring-orm-5.2.6.RELEASE
spring-tx-5.2.6.RELEASE
````
* 2、配置数据库连接池
````
<!-- 配置连接池-->
<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
    <property name="driverClassName" value="com.mysql.jdbc.Driver"></property>
    <property name="url" value="jdbc:mysql://127.0.0.1:3306/mysqlName"></property>
    <property name="username" value="root"></property>
    <property name="password" value="root"></property>
</bean>
````
* 3、配置JdbcTemplate对象，注入DataSource
````
<!--JdbcTemplate对象-->
<bean id="JdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <!--注入dataSource-->
    <property name="dataSource" ref="dataSource"></property>
</bean>
````
