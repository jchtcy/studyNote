# 一、OpenFeign简介
## 1、定义
````
Feign是一个声明式WebService客户端，使用Feign能让编写Web Service客户端更加简单。

它的使用方法是定义一个服务接口然后在上面添加注解，Feign也支持可拔插式的编码器和解码器，Spring Cloud对Feign进行了封装，使其支持了Spring MVC标准注解和HttpMessageConverters，Feign可以与Eureka和Ribbon组合使用以支持负载均衡。
````
## 2、作用
````
Feign旨在使编写Java Http客户端变得更容易，前面在使用Ribbon+RestTemplate时，利用RestTemplate 对 http请求的封装处理，形成了一套模版化的调用方法。但是在实际开发中，由于对服务依赖的调用可能不止一处，往往一个接口会被多处调用，所以通常都会针对每个微服务自行封装一些客户端类来包装这些依赖服务的调用。所以，Feign在此基础上做了进一步封装，由他来帮助我们定义和实现依赖服务接口的定义。

在Feign的实现下，我们只需创建一个接口并使用注解的方式来配置它(以前是Dao接口上面标注Mapper注解，现在是一个微服务接口上面标注一个Feign注解即可)，即可完成对服务提供方的接口绑定，简化了使用Spring cloud Ribbon时，自动封装服务调用客户端的开发量。

Feign集成了Ribbon，利用Ribbon维护了Payment的服务列表信息，并且通过轮询实现了客户端的负载均衡。而与Ribbon不同的是，通过feign只需要定义服务绑定接口且以声明式的方法，优雅而简单的实现了服务调用
````
## 3、Feign和OpenFeign两者区别
* 1、Feign
````
Feign是Spring Cloud组件中的一个轻量级RESTful的HTTP服务客户端
Feign内置了Ribbon，用来做客户端负载均衡，去调用服务注册中心的服务
Feign的使用方式是：使用Feign的注解定义接口，调用这个接口，就可以调用服务注册中心的服务

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-feign</artifactId>
</dependency>
````
* 2、OpenFeign
````
OpenFeign是Spring Cloud 在Feign的基础上支持了SpringMVC的注解，如@RequesMapping等等
OpenFeign 的 @FeignClient 可以解析 SpringMVC 的 @RequestMapping 注解下的接口，并通过动态代理的方式产生实现类，实现类中做负载均衡并调用其他服务

<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
````
# 二、OpenFeign使用步骤
## 1、新建cloud-consumer-feign-order80
## 2、pom.xml
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

    <artifactId>cloud-consumer-feign-order80</artifactId>
    <dependencies>
        <!-- Open Feign，他里面也有ribbon，所以有负载均衡 -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <!-- eureka Client -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <dependency>
            <!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
            <groupId>com.jch.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
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
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

</project>
````
## 3、application.yml
````
server:
  port: 80

eureka:
  client:
    register-with-eureka: false
    service-url: # 配置服务中心，openFeign去里面找服务
      defaultZone: http://eureka7001.com:7001/eureka/
````
## 4、主启动类
````
@SpringBootApplication
@EnableFeignClients
public class OrderFeginMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderFeginMain80.class,args);
    }
}
````
## 5、service层 
````
@RequestMapping对应调用服务端的访问地址
@FeignClient对应调用的服务端名称

@Component
@FeignClient(value = "CLOUD-PAYMENT-SERVICE")
public interface PaymentFeginService {
    @GetMapping(value = "/payment/get/{id}")
    public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id);
}
````
## 6、controller层
````
@RestController
@Slf4j
public class OrderFeginController {
    @Resource
    private PaymentFeginService paymentFeginService;

    @GetMapping(value = "/consumer/payment/get/{id}")
    public CommonResult<Payment> getPaymentById(@PathVariable("id") Long id){
        return paymentFeginService.getPaymentById(id);
    }
}
````
# 三、OpenFeign超时控制
## 1、服务提供端8001添加超时方法
````
@GetMapping("/payment/fegin/timeout")
public String paymentFeginTimeout(){
    try {
        TimeUnit.SECONDS.sleep(3);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    return serverPort;
}
````
## 2、服务消费方80添加超时方法
````
service层
@Component
@FeignClient(value = "cloud-payment-service")
public interface PaymentFeginService {
    @GetMapping("/payment/fegin/timeout")
    public String paymentFeginTimeout();
}
````
````
controller层
@RestController
@Slf4j
public class OrderFeginController {
    @Resource
    private PaymentFeginService paymentFeginService;
 
    @GetMapping("/consumer/payment/fegin/timeout")
    public String paymentFeginTimeout(){
        //openfegin-ribbon: 一般默认等待一秒,超过一秒报错
        return paymentFeginService.paymentFeginTimeout();
    }
}
````
## 3、此时服务消费端调用异常
## 4、 Openfeign默认超时等待为一秒，在消费方80配置超时时间
````
#设置feign客户端超时时间(OpenFeign默认支持ribbon)
ribbon:
  #指的是建立连接后从服务器读取到可用资源所用的时间
  ReadTimeout: 5000
  #指的是建立连接所用的时间，适用于网络状况正常的情况下,两端连接所用的时间
  ConnectTimeout: 5000
````
# 四、OpenFeign开启日志打印
````
Feign 提供了日志打印功能，我们可以通过配置来调整日志级别，从而了解 Feign 中 Http 请求的细节。

说白了就是对Feign接口的调用情况进行监控和输出
````
## 1、日志级别
````
NONE：默认的，不显示任何日志
BASIC：仅记录请求方法、URL、响应状态码及执行时间
HEADERS：除了 BASIC 中定义的信息之外，还有请求和响应的头信息
FULL：除了 HEADERS 中定义的信息之外，还有请求和响应的正文及元数据
````
## 2、添加config配置类
````
@Configuration
public class FeginConfig {
    @Bean
    Logger.Level feignLoggerLevel(){ // import feign.Logger;
        return Logger.Level.FULL;
    }
}
````
## 3、application.yml中开启日志打印
````
logging:
  level:
  # feign日志以什么级别监控哪个接口
    com.atguigu.springcloud.service.PaymentFeginService: debug
````