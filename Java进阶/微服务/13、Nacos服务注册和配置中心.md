# 一、SpringCloud Alibaba 简介
## 1、为什么会出现SpringCloud alibaba
````
Spring Cloud Netflix项目进入维护模式
````
* 1、什么是维护模式？
````
将模块置于维护模式，意味着 Spring Cloud 团队将不会再向模块添加新功能。我们将修复 block 级别的 bug 以及安全问题，我们也会考虑并审查社区的小型 pull request。
````
* 2、进入维护模式意味着什么呢？
````
进入维护模式意味着 Spring Cloud Netflix 将不再开发新的组件，我们都知道Spring Cloud 版本迭代算是比较快的，因而出现了很多重大ISSUE都还来不及Fix就又推另一个Release了。进入维护模式意思就是目前一直以后一段时间Spring Cloud Netflix提供的服务和功能就这么多了，不在开发新的组件和功能了。以后将以维护和Merge分支Full Request为主。

新组件功能将以其他替代平代替的方式实现
````
## 2、SpringCloud alibaba带来了什么
* 1、定义
````
官网: https://sca.aliyun.com/zh-cn/
诞生: 2018.10.31，Spring Cloud Alibaba 正式入驻了 Spring Cloud 官方孵化器，并在 Maven 中央库发布了第一个版本
````
* 2、作用
````
1、服务限流降级: 默认支持 Servlet、Feign、RestTemplate、Dubbo 和 RocketMQ 限流降级功能的接入，可以在运行时通过控制台实时修改限流降级规则，还支持查看限流降级 Metrics 监控。
2、服务注册与发现: 适配 Spring Cloud 服务注册与发现标准，默认集成了 Ribbon 的支持
3、分布式配置管理: 支持分布式系统中的外部化配置，配置更改时自动刷新。
4、消息驱动能力: 基于 Spring Cloud Stream 为微服务应用构建消息驱动能力。
5、阿里云对象存储: 阿里云提供的海量、安全、低成本、高可靠的云存储服务支持在任何应用、任何时间、任何地点存储和访问任意类型的数据。
5、分布式任务调度: 提供秒级、精准、高可靠、高可用的定时（基于 Cron 表达式）任务调度服务。同时提供分布式的任务执行模型，如网格任务。网格任务支持海量子任务均匀分配到所有 Worker（schedulerx-client）上执行。
6、阿里云短信服务: 覆盖全球的短信服务，友好、高效、智能的互联化通讯能力，帮助企业迅速搭建客户触达
````
* 3、下载
````
https://github.com/alibaba/spring-cloud-alibaba/blob/master/README-zh.md
````
* 4、实现
````
Sentinel
Nacos
RocketMQ
Dubbo
Seata
Alibaba Cloud OSS
Alibaba Cloud SchedulerX
Alibaba Cloud SMS
````
# 二、Nacos简介
## 1、为什么叫Nacos
````
前四个字母分别为Naming和Configuration的前两个字母，最后的s为Service

Nacos: Dynamic Naming and Configuration Service
````
## 2、定义
````
Nacos: Dynamic Naming and Configuration Service 一个更易于构建云原生应用的动态服务发现配置管理和服务管理平台

Nacos 就是注册中心 + 配置中心的组合 Nacos = Eureka+Config +Bus
````
## 3、作用
````
替代Eureka做服务注册中心
替代Config做服务配置中心
````
## 4、下载
````
https://github.com/alibaba/Nacos
````
## 5、各种注册中心比较
| 服务注册与发现框架 | CAP模型 | 控制台管理 | 社区活跃度 |
| :----:| :----: | :----: | :----: |
| Eureka | AP | 支持 | 低 |
| Zookeeper | CP | 不支持 | 中 |
| Consul | CP | 支持 | 高 |
| Nacos | AP | 支持 | 高 |
# 三、安装并运行Nacos
````
下载: https://github.com/alibaba/nacos/releases
单机启动: startup.cmd -m standalone
访问：http://localhost:8848/nacos 默认账号密码都是nacos 
````
# 四、Nacos作为服务注册中心演示
## 1、基于Nacos的服务提供者9001
* 1、父pom增加：
````
<dependency>
   <groupId>com.alibaba.cloud</groupId>
   <artifactId>spring-cloud-alibaba-dependencies</artifactId>
   <version>2.1.0.RELEASE</version>
   <type>pom</type>
   <scope>import</scope>
</dependency>
````
* 2、本模块pom： 
````
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>cloud2020</artifactId>
        <groupId>com.jch.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloudalibaba-providers-payment9001</artifactId>
    <dependencies>
        <!-- springcloud alibaba nacos 依赖 -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
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
        </dependency>
        <!-- 引入自己定义的api通用包 -->
        <dependency>
            <groupId>com.jch.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>

</project>
````
* 3、application.yml
````
server:
  port: 9001
spring:
  application:
    name: nacos-payment-provider
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
# springboot 监控 Actuator 的设置，暴露所有端点
management:
  endpoints:
    web:
      exposure:
        include: '*'
````
* 4、主启动类
````
@SpringBootApplication
@EnableDiscoveryClient
public class PaymentMain9001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain9001.class,args);
    }
}
````
* 5、controller层
````
@RestController
public class PaymentController {
    @Value("${server.port}")
    private String serverPort;

    @GetMapping(value = "/payment/nacos/{id}")
    public String getPayment(@PathVariable("id") Integer id) {
        return "nacos registry, serverPort: "+ serverPort+"\t id"+id;
    }
}
````
* 6、参照9001新建9002，新建cloudalibaba-provider-payment9002
## 2、基于Nacos的服务消费者83
* 1、pom.xml
````
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>cloud2020</artifactId>
        <groupId>com.jch.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloualibaba-orders-consumer83</artifactId>
    <dependencies>
        <!-- springcloud alibaba nacos 依赖 -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
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
        </dependency>
        <dependency><!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
            <groupId>com.jch.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>

</project>
````
* 2、application.yml 
````
server:
  port: 83
spring:
  application:
    name: nacos-order-consumer
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848

#消费者将要去访问的微服务名称(注册成功进nacos的微服务提供者)
service-url:
  nacos-user-service: http://nacos-payment-provider
````
* 3、主启动类
````
@SpringBootApplication
@EnableDiscoveryClient
public class OrderNacosMain83 {
    public static void main(String[] args) {
        SpringApplication.run(OrderNacosMain83.class,args);
    }
}
````
* 4、config类
````
@Configuration
public class ApplicationContextBean {
    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
````
* 5、controller层 
````
@RestController
public class OrderNacosController {

    @Value("${service-url.nacos-user-service}")
    private String serverURL;
    @Resource
    private RestTemplate restTemplate;

    @GetMapping(value = "/consumer/nacos/{id}")
    public String getPayment(@PathVariable Integer id) {
        return restTemplate.getForObject(serverURL + "/payment/nacos/" + id, String.class);
    }
}
````
* 6、为什么nacos支持负载均衡？
````
因为spring-cloud-starter-alibaba-nacos-discovery 导入了 ribbon
````
## 3、服务注册中心对比提升
![本地路径](img/Nacos与其他注册中心特性对比.png)
* 1、Nacos 支持AP和CP模式的切换
````
C是所有节点在同一时间看到的数据是一致的；而A的定义是所有的请求都会收到响应
````
* 2、何时选择使用何种模式？
````
一般来说，如果不需要存储服务级别的信息且服务实例是通过nacos-client注册，并能够保持心跳上报，那么就可以选择AP模式。当前主流的服务如 Spring cloud 和 Dubbo 服务，都适用于AP模式，AP模式为了服务的可能性而减弱了一致性，因此AP模式下只支持注册临时实例。

如果需要在服务级别编辑或者存储配置信息，那么 CP 是必须，K8S服务和DNS服务则适用于CP模式。CP模式下则支持注册持久化实例，此时则是以 Raft 协议为集群运行模式，该模式下注册实例之前必须先注册服务，如果服务不存在，则会返回错误。
````
* 3、切换命令
````
curl -X PUT '$NACOS_SERVER:8848/nacos/v1/ns/operator/switches?entry=serverMode&value=CP'
````
# 五、Nacos作为服务配置中心演示
## 1、Nacos作为配置中心3377-基础配置
* 1、pom.xml
````
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>cloud2020</artifactId>
        <groupId>com.jch.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloudalibaba-config-nacos-client3377</artifactId>
    <dependencies>

        <!-- 以 nacos 做服务配置中心的依赖 -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>

        <!-- springcloud alibaba nacos 依赖 -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!-- springboot整合Web组件 -->
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
        </dependency>
        <dependency><!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
            <groupId>com.jch.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
    </dependencies>
</project>
````
* 2、bootstrap.yml
````
Nacos 同 springcloud-config 一样，在项目初始化时，要保证先从配置中心进行配置拉取，拉取配置之后，才能保证项目的正常启动。
springboot 中配置文件的加载是存在优先级顺序的，bootstrap 优先级高于application
````
````
server:
  port: 3377

spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #Nacos服务注册中心地址
      config:
        server-addr: localhost:8848 #Nacos作为配置中心地址
        file-extension: yaml #指定yaml格式的配置
````
* 3、application.yml
````
spring:
  profiles:
    active: dev # 表示开发环境
````
* 4、主启动类
````
@SpringBootApplication
@EnableDiscoveryClient
public class NacosConfigMain3377 {
    public static void main(String[] args) {
        SpringApplication.run(NacosConfigMain3377.class,args);
    }
}
````
* 5、controller层
````
@RestController
@RefreshScope //支持Nacos的动态刷新功能
public class ConfigClientController {
    @Value("${config.info}")
    private String configInfo;

    @GetMapping("/config/info")
    public String getConfigInfo(){
        return configInfo;
    }
}
````
## 2、在Nacos中添加配置信息
* 1、理论
````
nacos中的dataid的组成格式及与springboot配置文件中的匹配规则:
${prefix}-${spring.profiles.active}.${file-extension}

prefix 默认为 spring.application.name 的值，也可以通过配置项 spring.cloud.nacos.config.prefix 来配置
spring.profiles.active 即为当前环境对应的 profile，详情可以参考 Spring Boot文档。 注意：当 spring.profiles.active 为空时，对应的连接符 - 也将不存在，dataId的拼接格式变成 ${prefix}.${file-extension}
file-exetension 为配置内容的数据格式，可以通过配置项 spring.cloud.nacos.config.file-extension 来配置。目前只支持 properties 和 yaml类型(注意nacos里必须使用yaml)

最后公式: ${spring.application.name}-${spring.profiles.active}.${spring.cloud.nacos.config.file-extension}
````
* 2、实操
````
配置新增：nacos-config-client-dev.yaml

Data ID: nacos-config-client-dev.yaml
Group: DEFAULT_GROUP
配置格式: YAML
配置内容: 
config:
    info: "config info for dev, from nacos config center."
````
* 3、测试
````
1、启动前需要在nacos客户端-配置管理-配置管理栏目下有对应的yaml配置文件
2、修改nacos中的yaml配置文件, 再次调用查看配置的接口, 就会发现配置已经刷新
````
## 3、Nacos作为配置中心-分类配置
* 1、问题——多环境多项目管理
````
问题1：
实际开发中，通常一个系统会准备：dev开发环境、test测试环境、prod生产环境，如何保证指定环境启动时服务能正确读取到Nacos上相应环境的配置文件呢？

问题2：
一个大型分布式微服务系统会有很多微服务子项目，每个微服务项目又都会有相应的开发环境、测试环境、预发环境、正式环境…那怎么对这些微服务配置进行管理呢？
````
* 2、Nacos的图形化管理界面
````
命名空间可以新建 prod、dev、test
````
* 3、Namespace+Group+Data ID三者关系?
![本地路径](img/Namespace+Group+Data%20ID三者关系.png)
````
Namespace=public，Group=DEFAULT_GROUP，默认Cluster是DEFAULT

Nacos默认的命名空间是public；Namespace主要用来实现隔离
    比方说我们现在有三个环境：开发、测试、生产环境，我们就可以创建三个Namespace，不同的Namespace之间是隔离的

Group默认是DEFAULT_GROUP，Group可以把不同的微服务划分到同一个分组里面去

Service就是微服务；一个Service可以包含多个Cluster（集群），Nacos默认Cluster是DEFAULT，Cluster是对指定微服务的一个虚拟划分。
比方说为了容灾，将Service微服务分别部署在了杭州机房和广州机房，这时就可以给杭州机房的Service微服务起一个集群名称（HZ），给广州机房的Service微服务起一个集群名称（GZ），还可以尽量让同一个机房的微服务互相调用，以提升性能。
最后是Instance，就是微服务的实例
````
## 4、三种方案加载配置 Case
* 1、 DataID方案(就是nacos的文件名)
````
新建test配置DataID：nacos-config-client-test.yaml
config:
    info: "config info for test, from nacos config center."
````
````
spring:
  profiles:
    active: test # 测试环境
````
* 2、 Group方案(默认DEFAULT_GROUP)
````
新建 DEV_GROUP
Data ID: nacos-config-client-info.yaml
Group: DEV_GROUP
描述: DEV_GROUP config
配置格式: yaml
配置内容: 
config:
    info: "nacos-config-client-info.yaml, DEV_GROUP"
````
````
新建 TEST_GROUP
Data ID: nacos-config-client-info.yaml
Group: TEST_GROUP
描述: TEST_GROUP config
配置格式: yaml
配置内容: 
config:
    info: "nacos-config-client-info.yaml, TEST_GROUP"
````
````
bootstrap.yml

server:
  port: 3377

spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #Nacos服务注册中心地址
      config:
        server-addr: localhost:8848 #Nacos作为配置中心地址
        file-extension: yaml #指定yaml格式的配置
        group: DEV_GROUP
#        group: TEST_GROUP
````
````
application.yml

spring:
  profiles:
    active: info
````
* 3、Namespace方案(默认public)
````
新建 dev和test两个命名空间, ID自动生成
````
````
在dev命名空间下新建三个配置文件

Data ID: nacos-config-client-dev.yaml
Group: DEFAULT_GROUP
配置格式: yaml
配置内容: 
config:
    info: "e7a0b658-110f-49d7-aa55-cf1c95358350 dev DEFAULT_GROUP"

Data ID: nacos-config-client-dev.yaml
Group: DEV_GROUP
配置格式: yaml
配置内容: 
config:
    info: "e7a0b658-110f-49d7-aa55-cf1c95358350 dev DEV_GROUP"

Data ID: nacos-config-client-dev.yaml
Group: TEST_GROUP
配置格式: yaml
配置内容: 
config:
    info: "e7a0b658-110f-49d7-aa55-cf1c95358350 dev TEST_GROUP"
````
````
bootstrap.yml

server:
  port: 3377

spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848 #Nacos服务注册中心地址
      config:
        server-addr: localhost:8848 #Nacos作为配置中心地址
        file-extension: yaml #指定yaml格式的配置
        group: DEV_GROUP
        namespace: e7a0b658-110f-49d7-aa55-cf1c95358350
````
````
application.yml

spring:
  profiles:
    active: dev
````
# 六、Nacos集群和持久化配置(重要)
## 1、官网说明
````
https://nacos.io/zh-cn/docs/cluster-mode-quick-start.html
````
## 2、Nacos持久化配置解释(windows)
````
Nacos默认自带的是嵌入式数据库derby

derby到mysql切换配置步骤：
    nacos-server-1.1.4\nacos\conf目录下找到sql脚本，执行 nacos-mysql.sql
    修改nacos/conf/application.properties文件(切换数据库)，增加支持mysql数据源配置（目前只支持mysql），添加mysql数据源的url、用户名和密码

### If use MySQL as datasource:
spring.datasource.platform=mysql

### Count of DB:
db.num=1

### Connect URL of DB:
db.url.0=jdbc:mysql://127.0.0.1:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
db.user.0=root
db.password.0=root

再以单机模式启动nacos(重启)，nacos所有写嵌入式数据库的数据都写到了mysql
单机的数据库都是独立的，我们得让他们共用一个数据库
````
## 3、Nacos集群配置(linux)
* 1、Linux服务器上mysql数据库配置
````
创建数据库nacos_config
执行sql脚本 nacos/conf/nacos-mysql.sql
````
* 2、application.properties配置
````
### If use MySQL as datasource:
spring.datasource.platform=mysql

### Count of DB:
db.num=1

### Connect URL of DB:
db.url.0=jdbc:mysql://127.0.0.1:3306/nacos_config?characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useUnicode=true&useSSL=false&serverTimezone=UTC
db.user.0=root
db.password.0=root
````
* 3、Linux服务器上nacos的集群配置cluster.conf
````
# 告诉这3个集群结点是一组的 # 不能写127.0.0.1，必须是linux hostname -i能够识别的ip
192.168.1.2:3333
192.168.1.2:4444
192.168.1.2:5555
````
* 4、编辑Nacos的启动脚本startup.sh, 使它能够接收不同的启动端口
````
# 第一处修改, 接收参数由三个变为四个
while getopts ": m: f: s: p:" opt
do
  case $opt in
    m)
      MODE=$OPTARG;;
    f)
      FUNCTION_MODE=$OPTARG;;
    s)
      SERVER=$OPTARG;;
    p)
      PORT=$OPTARG;;
    ?)
    echo "Unknown parameter"
    exit 1;;
  esac
done
````
````
# 第二处修改, 启动时通过传入的参数指定端口号
# start
nohup $JAVA -Dserver.port=${PORT} ${JAVA_OPT} nacos.nacos >> ${BASE_DIR}/logs/start.out 2>&1 &
````
* 5、Ngnix的配置, 由它作负载均衡
* 6、application.yml
````
server-addr: 192.168.111.144:1111 #换成nginx的1111端口, 做集群
````
## 4、高可用小总结
![本地路径](img/高可用小总结.png)