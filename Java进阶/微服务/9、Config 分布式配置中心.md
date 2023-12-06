# 一、Config分布式配置中心介绍
## 1、分布式系统面临的配置问题
````
微服务意味着要将单体应用中的业务拆分成一个个子服务，每个服务的粒度相对较小，因此系统中会出现大量的服务。由于每个服务都需要必要的配置信息才能运行，所以一套集中式的、动态的配置管理设施是必不可少的。

SpringCloud提供了ConfigServer来解决这个问题，我们每一个微服务自己带着一个application.yml，上百个配置文件的管理
````
## 2、定义
````
SpringCloud Config为微服务架构中的微服务提供集中化的外部配置支持，配置服务器为各个不同微服务应用的所有环境提供了一个中心化的外部配置。
````
## 3、实现
````
SpringCloud Config分为 服务端 和 客户端 两部分。

1、服务端也称为分布式配置中心，它是一个独立的微服务应用，用来连接配置服务器并为客户端提供获取配置信息，加密/解密信息等访问接口
2、客户端则是通过指定的配置中心来管理应用资源，以及与业务相关的配置内容，并在启动的时候从配置中心获取和加载配置信息,配置服务器默认采用git来存储配置信息，这样就有助于对环境配置进行版本管理，并且可以通过git客户端工具来方便的管理和访问配置内容。
````
## 4、作用
````
集中管理配置文件
不同环境不同配置，动态化的配置更新，分环境部署比如dev/test/prod/beta/release
运行期间动态调整配置，不再需要在每个服务部署的机器上编写配置文件，服务会向配置中心统一拉取配置自己的信息
当配置发生变动时，服务不需要重启即可感知到配置的变化并应用新的配置
将配置信息以REST接口的形式暴露：post、curl访问刷新均可…
````
# 二、Config配置总控中心搭建
## 1、服务端远程仓库配置
* 1、新建远程仓库
````
在Gitee上新建一个名为springcloud-config的新Repository

由上一步获得刚新建的git地址

https://gitee.com/jch/springcloud-config.git
````
* 2、在仓库下新建三个文件
````
config-dev.yml
config:
    info: master branch, springcloud-config/config-dev.yml version=1

config-prod.yml
config:
    info: master branch, springcloud-config/config-prod.yml version=1

config-test.yml
config:
    info: master branch, springcloud-config/config-test.yml version=1
````
## 2、配置模块 cloud-config-center-3344，它即为Cloud的配置中心模块
* 1、pom.xml
````
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>cloud2020</artifactId>
        <groupId>springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
 
    <artifactId>cloud-config-center-3344</artifactId>
 
    <dependencies>
        <!-- config Server -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
 
        <!--eureka-client config Server也要注册进服务中心-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency><!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
            <groupId>springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>
 
</project>
````
* 2、application.yml
````
server:
  port: 3344
 
spring:
  application:
    name: cloud-config-center #注册进eurkea服务器的微服务名
  cloud:
    config:
      server:
        git:
          uri: https://gitee.com/jch/springcloud-config.git #gitee上面的仓库名
          # 搜索目录
          search-paths:
            - springcloud-config
        # 读取分支
        label: master
 
# 服务注册到eureka
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka
````
* 3、主启动类
````
@SpringBootApplication
@EnableConfigServer
public class ConfigCenterMain3344{
    public static void main(String[] args) {
            SpringApplication.run(ConfigCenterMain3344.class, args);
    }
}
````
## 3、文件命名规则
````
{application} 是应用名称，对应到配置文件上来，就是配置文件的名称部分。

{profile} 是配置文件的版本，我们的项目有开发版本、测试环境版本、生产环境版本，对应到配置文件上来就是以 application-{profile}.yml 加以区分，例如application-dev.yml，application-prod.yml。
````
## 4、文件访问规则
````
/{label}/{application}-{profile}.yml

不加分支默认是master
/{application}-{profile}.yml

/{application}/{profile}[/{label}] 输出json字符串
````
# 三、Config客户端配置与测试
## 1、pom.xml
````
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>cloud2020</artifactId>
        <groupId>springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
 
    <artifactId>cloud-config-client-3355</artifactId>
 
    <dependencies>
    <!-- config Client 和 服务端的依赖不一样 -->
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-config</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-devtools</artifactId>
        <scope>runtime</scope>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <optional>true</optional>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    <!-- 引入自己定义的api通用包 -->
    </dependency>
        <dependency>
            <groupId>springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>
 
</project>
````
## 2、bootstrap.yml
````
appllication.yml是用户级的资源配置项
bootstrap.yml是系统级的，优先级更加高
SpringCloud会创建一个"Bootstrap Context"作为Spring应用的ApplicationContext的父上下文。

初始化的时候，Bootstrap Context负责从外部源加载配置属性并解析配置。这两个上下文共享一个从外部获取的Environment。

Bootstrap 属性有高优先级，默认情况下，它们不会被本地配置覆盖。BootstrapContext和ApplicationContext、有着不同的约定，所以新增了一个bootstrap.yml文件，保证BootstrapContext和ApplicationContext配置的分离。
````
````
bootstrap.yml
server:
  port: 3355
 
spring:
  application:
    name: config-client
  cloud:
    #Config客户端配置
    config:
      label: master #分支名称
      name: config #配置文件名称
      profile: test #读取后缀名称   上述3个综合：master分支上config-test.yml的配置文件被读取
      uri: http://localhost:3344 #配置中心地址
      # 综合上面四个 即读取配置文件地址为： http://localhost:3344/master/config-test.yml
 
#服务注册到eureka地址
eureka:
  client:
    service-url:
      defaultZone: http://localhost:7001/eureka
````
## 3、主启动类
````
@SpringBootApplication
@EnableEurekaClient
public class ConfigClientMain3355{
        public static void main(String[] args) {
            SpringApplication.run(ConfigClientMain3355.class, args);
        }
}
````
## 4、 controller层
````
@RestController
public class ConfigClientController {
    @Value("${config.info}") //gitee里的yml文件里的内容
    private String configInfo;
 
    @RequestMapping("/configInfo")
    public String getConfigInfo(){
        return configInfo;
    }
}
````
## 5、修改远程仓库配置文件内容
````
修改config-dev.yml文件内容，其余两个类似
spring:
  application:
    name: springcloud-config-dev
  profiles:
    active: dev
config:
    info: master branch, springcloud-config/config-dev.yml version=2
````
## 6、存在问题
````
当远程仓库配置文件被修改后，3344和3355都要变，3344变了，但是3355必须要重启才可以更新。
````
# 四、Config客户端之动态刷新
## 1、pom.xml引入actuator监控
````
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
````
## 2、修改YML，暴露监控端口
````
# 暴露监控端点 actuator刷新配置
management:
  endpoints:
    web:
      exposure:
        include: "*"
````
## 3、Controller类添加@RefreshScope
````
@RestController
@RefreshScope
public class ConfigClientController {
    @Value("${config.info}") //gitee里的yml文件里的内容
    private String configInfo;
 
    @RequestMapping("/configInfo")
    public String getConfigInfo(){
        return configInfo;
    }
}
````
## 4、问题和解决
````
此时修改配置文件内容，再测：http://localhost:3355/configInfo，发现3355还是没有变化

需要运维人员发送Post请求刷新3355
curl -X POST "http://localhost:3355/actuator/refresh"
````
## 5、思考
````
想想还有什么问题？

    假如有多个微服务客户端3355/3366/3377......，每个微服务都要执行一次post请求，手动刷新？
    可否广播，一次通知，处处生效？
    我们想大范围的自动刷新，求方法
于是就有下面的消息总线！
````