# 一、Sentinel介绍
## 1、定义
````
随着微服务的流行，服务和服务之间的稳定性变得越来越重要。 Sentinel 以流量为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。

Sentinel 具有以下特征:
    1、丰富的应用场景：Sentinel 承接了阿里巴巴近 10 年的双十一大促流量的核心场景，例如秒杀（即突发流量控制在系统容量可以承受的范围）、消息削峰填谷、集群流量控制、实时熔断下游不可用应用等。
    2、完备的实时监控：Sentinel 同时提供实时的监控功能。您可以在控制台中看到接入应用的单台机器秒级数据，甚至 500 台以下规模的集群的汇总运行情况。
    3、广泛的开源生态：Sentinel 提供开箱即用的与其它开源框架/库的整合模块，例如与 Spring Cloud、Dubbo、gRPC 的整合。您只需要引入相应的依赖并进行简单的配置即可快速地接入 Sentinel。
    4、完善的 SPI 扩展点：Sentinel 提供简单易用、完善的 SPI 扩展接口。您可以通过实现扩展接口来快速地定制逻辑。例如定制规则管理、适配动态数据源等。
````
## 2、Hystrix与Sentinel比较
````
Hystrix：
    1、需要我们程序员自己手工搭建监控平台
    2、没有一套web界面可以给我们进行更加细粒度化得配置流控、速率控制、服务熔断、服务降级

Sentinel：
    1、单独一个组件，可以独立出来。
    2、支持界面化的细粒度统一配置。

约定 > 配置 > 编码
````
## 3、服务使用中的各种问题
````
服务雪崩
服务降级
服务熔断
服务限流
````
## 4、Sentinel 分为两个部分：
````
核心库（Java 客户端）不依赖任何框架/库，能够运行于所有 Java 运行时环境，同时对 Dubbo / Spring Cloud 等框架也有较好的支持。
控制台（Dashboard）基于 Spring Boot 开发，打包后可以直接运行，不需要额外的 Tomcat 等应用容器。
````
# 二、Sentinel下载安装运行
* 1、下载
````
https://github.com/alibaba/Sentinel/releases
````
* 2、运行命令
````
前提：java8环境OK + 8080端口不能被占用

运行命令: 指定端口8888
java -jar sentinel-dashboard-1.8.6.jar --server.port=8888

http://localhost:8888，登录账号密码均为sentinel
````
# 三、初始化演示工程
## 新建 cloudalibaba-sentinel-service8401
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

    <artifactId>cloudalibaba-sentinel-service8401</artifactId>

    <dependencies>
        <!--         后续做Sentinel的持久化会用到的依赖-->
        <dependency>
            <groupId>com.alibaba.csp</groupId>
            <artifactId>sentinel-datasource-nacos</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>
        <!-- springcloud alibaba nacos 依赖,Nacos Server 服务注册中心 -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!--openfeign-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
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

        <!-- 日常通用jar包 -->
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
  port: 8401
spring:
  application:
    name: cloudalibaba-sentinel-service
  cloud:
    nacos:
      discovery:
        # 服务注册中心 # sentinel注册进nacos
        server-addr: localhost:8848
    sentinel:
      transport:
        # 配置 Sentinel Dashboard 的地址
        dashboard: localhost:8888
        # 默认8719 ，如果端口被占用，端口号会自动 +1，直到找到未被占用的端口，提供给 sentinel 的监控端口
        port: 8719

management:
  endpoints:
    web:
      exposure:
        include: '*'
````
* 3、主启动类
````
@SpringBootApplication
@EnableDiscoveryClient
public class MainApp8401 {
    public static void main(String[] args) {
        SpringApplication.run(MainApp8401.class,args);
    }
}
````
* 4、controller层
````
@RestController
@Slf4j
public class FlowLimitController {
    @GetMapping("/testA")
    public String testA() {
        return "------testA";
    }

    @GetMapping("/testB")
    public String testB() {
        log.info(Thread.currentThread().getName() + "\t" + "...testB");
        return "------testB";
    }
}
````
* 5、测试
````
访问Sentinel无数据

Sentinel采用的懒加载说明：
执行一次访问即可：http://localhost:8401/testA、http://localhost:8401/testB
````
# 四、流控规则
## 1、定义
* 1、名词解释
````
1、资源名：唯一名称，默认请求路径
2、针对来源：sentinel可以针对调用者进行限流，填写微服务名，默认default（不区分来源）
3、阈值类型/单机值：
    QPS(每秒钟的请求数量)：当调用该api就QPS达到阈值的时候，进行限流
    线程数．当调用该api的线程数达到阈值的时候，进行限流
4、是否集群：不需要集群
5、流控模式：
    直接：api达到限流条件时，直接限流。分为QPS和线程数
    关联：当关联的资源达到阈值时，就限流自己。
    链路：只记录指定链路上的流量(指定资源从入口资源进来的流量，如果达到阈值，就进行限流)[api级别的针对来源]
6、流控效果：
    快速失败：直接抛异常
    warm up：根据codeFactor（冷加载因子，默认3）的值，从阈值codeFactor，经过预热时长，才达到设置的QPS阈值
    排队等待：匀速排毒，让请求以匀速通过，阈值类型必须设置为QPS，否则无效
````
* 2、QPS和线程数的区别
````
QPS 类似于银行的保安：所有的请求到Sentinel 后，他会根据阈值放行，超过报错

线程数类似于银行的窗口：所有的请求会被放进来，但如果阈值设置为1，那么其他的请求就会报错也就是，银行里只有一个窗口，一个人在办理业务中，其他人跑过来则会告诉你“现在不行，没到你”
````
* 3、重要属性

| Field | 说明 | 默认值 |
| :----:| :----: | :----: |
| resource | 资源名，资源名是限流规则的作用对象 |  |
| count | 限流阈值 |  |
| grade  | 限流阈值类型，QPS 模式(1)或并发线程数模式(0) | QPS 模式 |
| limitApp | 流控针对的调用来源 | default，代表不区分调用来源 |
| strategy | 调用关系限流策略：直接、链路、关联 | 根据资源本身(直接) |
| controlBehavior  | 流控效果（直接拒绝/WarmUp/匀速+排队等待），不支持按调用关系限流 | 直接拒绝 |
| clusterMode | 是否集群限流 | 否 |
## 2、流控模式
* 1、直接（默认）
````
直接->快速失败：系统默认

配置及说明：表示1秒钟内查询1次就是OK，若超过次数1，就直接-快速失败，报默认错误

测试：快速点击访问：http://localhost:8401/testA
````
* 2、关联
````
当关联的资源达到阈值时，就限流自己

设置testA的关联资源为testB
当与A关联的资源B达到阀值后，就限流A自己

postman里新建多线程集合组来测试
````
* 3、链路
````
只记录指定链路上的流量(指定资源从入口资源进来的流量，如果达到阈值，就进行限流)[API级别的针对来源]
````
````
例如有两条请求链路
/testA -> /common
/testB -> /common

如果只希望限制从testB进入到/common的请求
资源名: /common
流控模式: 链路
入口资源: /test2
````
## 3、流控效果
* 1、快速失败（默认的流控处理）
````
直接失败，抛出异常：Blocked by Sentinel (flow limiting)
````
* 2、预热
````
1、定义
Warm Up（RuleConstant.CONTROL_BEHAVIOR_WARM_UP）方式，即预热/冷启动方式。当系统长期处于低水位的情况下，当流量突然增加时，直接把系统拉升到高水位可能瞬间把系统压垮。通过"冷启动"，让通过的流量缓慢增加，在一定时间内逐渐增加到阈值上限，给冷系统一个预热的时间，避免冷系统被压垮。

2、WarmUp配置
默认 coldFactor 为 3，即请求QPS从(threshold / 3) 开始，经多少预热时长才逐渐升至设定的 QPS 阈值
coldFactor 冷加载因子, threshold QPS阈值

3、例子
阈值类型: QPS 单机阈值: 10
流控效果: Warm Up
预热时长: 5

以上配置表示: 
初始单机阈值为(10 / 3) = 3, 经过5秒预热时长, 提升阈值到10

4、应用场景
秒杀系统在开启的瞬间，会有很多流量上来，很有可能把系统打死，预热方式就是把为了保护系统，可慢慢的把流量放进来，慢慢的把阀值增长到设置的阀值
````
* 4、排队等待
````
1、定义
匀速排队，让请求以均匀的速度通过，阀值类型必须设成QPS，否则无效

2、例子
资源名: testA
针对来源: default
阈值类型: QPS 单机阈值: 1
流控效果: 排队等待
超时时间: 20000

设置含义：/testA每秒1次请求，超过的话就排队等待，等待的超时时间为20000毫秒
匀速排队，阈值必须设置为QPS 

匀速排队（RuleConstant.CONTROL_BEHAVIOR_RATE_LIMITER）方式会严格控制请求通过的间隔时间，也即是让请求以均匀的速度通过，对应的是漏桶算法。
````
# 五、降级规则 
## 1、熔断降级概述
````
除了流量控制以外，对调用链路中不稳定的资源进行熔断降级也是保障高可用的重要措施之一。一个服务常常会调用别的模块，可能是另外的一个远程服务、数据库，或者第三方 API 等。例如，支付的时候，可能需要远程调用银联提供的 API；查询某个商品的价格，可能需要进行数据库查询。然而，这个被依赖服务的稳定性是不能保证的。如果依赖的服务出现了不稳定的情况，请求的响应时间变长，那么调用服务的方法的响应时间也会变长，线程会产生堆积，最终可能耗尽业务自身的线程池，服务本身也变得不可用。

现代微服务架构都是分布式的，由非常多的服务组成。不同服务之间相互调用，组成复杂的调用链路。以上的问题在链路调用中会产生放大的效果。复杂链路上的某一环不稳定，就可能会层层级联，最终导致整个链路都不可用。因此我们需要对不稳定的弱依赖服务调用进行熔断降级，暂时切断不稳定调用，避免局部不稳定因素导致整体的雪崩。熔断降级作为保护自身的手段，通常在客户端（调用端）进行配置。
````
## 2、降级策略介绍
````
1、RT(平均响应时间，秒级)
    平均响应时间超出阈值且在时间窗口内通过的请求>=5，两个条件同时满足后触发降级
    窗口期过后关闭断路器
    RT最大4900(更大的需要通过-Dcsp.sentinel.statistic.max.rt=XXXX才能生效)

2、异常比列(秒级)
    QPS >= 5且异常比例（秒级统计）超过阈值时，触发降级；时间窗口结束后，关闭降级

3、异常数(分钟级)
    异常数(分钟统计)超过阈值时，触发降级；时间窗口结束后，关闭降级
````
## 3、RT
````
平均响应时间(DEGRADE_GRADE_RT)：当1s内持续进入5个请求，对应时刻的平均响应时间（秒级）均超过阈值（ count，以ms为单位），那么在接下的时间窗口（DegradeRule中的timeWindow，以s为单位）之内，对这个方法的调用都会自动地熔断(抛出DegradeException )。注意Sentinel 默认统计的RT上限是4900 ms，超出此阈值的都会算作4900ms，若需要变更此上限可以通过启动配置项-Dcsp.sentinel.statistic.max.rt=xxx来配置。
注意：Sentinel 1.7.0才有平均响应时间（DEGRADE_GRADE_RT），Sentinel 1.8.0的没有这项，取而代之的是慢调用比例 (SLOW_REQUEST_RATIO)。
````
## 4、慢调用比例
* 1、定义
````
慢调用比例 (SLOW_REQUEST_RATIO)：选择以慢调用比例作为阈值，需要设置允许的慢调用 RT（即最大的响应时间），请求的响应时间大于该值则统计为慢调用。当单位统计时长（statIntervalMs）内请求数目大于设置的最小请求数目，并且慢调用的比例大于阈值，则接下来的熔断时长内请求会自动被熔断。经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求响应时间小于设置的慢调用 RT 则结束熔断，若大于设置的慢调用 RT 则会再次被熔断
````
* 2、例子
````
controller

@RestController
@Slf4j
public class FlowLimitController {
    @GetMapping("/testA")
    public String testA() {
        try {
            TimeUnit.SECONDS.sleep(1);//睡眠1秒
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "------testA";
    }
}
````
````
新增熔断规则

资源名: /testA
熔断策略: 慢调用比例
最大RT: 800
比例阈值: 1
熔断时长: 20
最小请求数: 5
统计时长: 1000

说明: 在1000毫秒内，当请求数大于5，接口响应时间大于800ms，慢调用比例达到100%时，接下来的20秒内自动开启熔断。
````
## 5、异常比例
* 1、定义
````
异常比例：当单位统计时长（statIntervalMs）内请求数大于设置的最小请求数，并且异常比例大于设置的比例阈值，则接下来的熔断时长内请求自动会被熔断。经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求成功（无异常）则结束熔断，否则会再次被熔断。
````
* 2、例子
````
controller

@RestController
@Slf4j
public class FlowLimitController {
    @GetMapping("/testA")
    public String testA(Integer id) {
        if(id != null && id < 0){
            throw new RuntimeException("运行异常");
        }
        return "------testA";
    }
}
````
````
新增熔断规则

资源名: /testA
熔断策略: 异常比例
比例阈值: 0.5
熔断时长: 20
最小请求数: 5
统计时长: 1000

说明: 在1000毫秒内，当请求数大于5且异常比例达到阈值50%时，接下来的20秒内自动开启熔断。
````
## 6、异常数
* 1、定义
````
当单位统计时长（statIntervalMs）内请求数大于设置的最小请求数，并且异常数大于设置的异常数，则接下来的熔断时长内请求自动会被熔断。经过熔断时长后熔断器会进入探测恢复状态（HALF-OPEN 状态），若接下来的一个请求成功（无异常）则结束熔断，否则会再次被熔断。
````
* 2、例子
````
controller

@RestController
@Slf4j
public class FlowLimitController {
    @GetMapping("/testA")
    public String testA() {
        try {
            TimeUnit.SECONDS.sleep(1);//睡眠1秒
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        return "------testA";
    }
}
````
````
新增熔断规则

资源名: /testA
熔断策略: 异常数
比例阈值: 5
熔断时长: 20
最小请求数: 5
统计时长: 1000

说明: 在1000毫秒内，当请求数大于5且异常数达到5时，接下来的20秒内自动开启熔断。
````
# 六、热点key限流
## 1、定义
````
何为热点：热点即经常访问的数据，很多时候我们希望统计或者限制某个热点数据中访问频次最高的 Top N 数据，并对其访问进行限流或者其它操作：
    商品 ID 为参数，统计一段时间内最常购买的商品 ID 并进行限制
    用户 ID 为参数，针对一段时间内频繁访问的用户 ID 进行限制

热点参数限流会统计传入参数中的热点参数，并根据配置的限流阈值与模式，对包含热点参数的资源调用进行限流。热点参数限流可以看做是一种特殊的流量控制，仅对包含热点参数的资源调用生效。
````
## 2、例子
* 1、controller
````
@RestController
@Slf4j
public class FlowLimitController {
    @GetMapping("/testHotKey")
    @SentinelResource(value = "testHotKey", blockHandler = "deal_testHotKey")
    public String testHotKey(@RequestParam(value = "p1", required = false) String p1,
                             @RequestParam(value = "p2", required = false) String p2) {
        return "------testHotKey------";
    }
    
    public String deal_testHotKey(String p1, String p2, BlockException exception) {
        return "deal_testHotKey";
    }
}
````
* 2、新增热点规则
````
资源名: testHotKey
限流模式: QPS模式
参数索引: 0
单机阈值: 1
统计窗口时长: 1

资源名为 @SentinelResource注解的value属性
参数索引 @SentinelResource(value = "testHotKey")注解的方法的第几个参数, 0代表第一个参数，1代表第二个参数
````
## 3、参数例外项
````
特例情况：我们期望p1参数当它是某个特殊值时，它的限流值和平时不一样

特例：假如当p1的值等于5时，它的阈值可以达到200

热点参数的注意点，参数必须是基本类型或者String
````
````
资源名: testHotKey
限流模式: QPS模式
参数索引: 0
单机阈值: 1
统计窗口时长: 1

参数例外项
参数值: 5    限流阈值: 200

资源名为 @SentinelResource注解的value属性
参数索引 @SentinelResource(value = "testHotKey")注解的方法的第几个参数, 0代表第一个参数，1代表第二个参数
当参数值为5时, 限流阈值为200
````
````
注意
@SentinelResource ：处理的是Sentine控制台配置的违规情况，有blockHandler方法配置的兜底处理
@RuntimeException：int age=10/0，这个是java运行时报出的运行时异异常RunTimeException，@SentineResource不管
````
# 七、系统规则
````
1、Load自适应(仅对 Linux/Unix-like 机器生效): 系统的load1作为启发指标, 进行自适应系统保护。当系统load1超过设定的启发值, 且系统当前的并发线程数超过估算的系统容量时才会触发系统保护(BBR阶段)。系统容量由系统的maxQos * minRt估算得出。设定参考值一般是CPU cores * 2.5
2、CPU usage: 当系统CPU使用率超过阈值即触发系统保护(取值范围0-1), 比较灵敏
3、平均RT: 当单台机器上所有入口流量的平均RT达到阈值即触发系统保护, 单位时毫秒
4、并发线程数: 当单台机器上所有入口流量的并发线程数达到阈值即触发系统保护
5、入口QPS: 当单台机器上所有入口流量的QPS达到阈值即触发系统保护
````
# 八、@SentinelResource配置
## 1、按资源名称限流+后续处理
* 1、controller
````
@RestController
public class RateLimitController {

    @GetMapping("/byResource")
    @SentinelResource(value = "byResource", blockHandler = "handleException")
    public CommonResult byResource(){
        return new CommonResult(200, "按资源名称限流测试OK", new Payment(2020L, "serial001"));
    }
    public CommonResult handleException(BlockException exception){
        return new CommonResult(444, exception.getClass().getCanonicalName() + "\t 服务不可用");
    }
}
````
* 2、新增流控规则
````
资源名: byResource
阈值类型: QPS 单机阈值: 1

表示1秒钟内查询次数大于1，就跑到我们自定义的兜底方法
````
* 3、额外问题
````
此时关闭服务8401：查看Sentinel控制台，流控规则消失
````
## 2、按照Url地址限流+后续处理
* 1、通过访问的URL来限流，会返回Sentinel自带默认的限流处理信息
````
RateLimitController新增
@GetMapping("/rateLimit/byUrl")
@SentinelResource(value = "byUrl")
public CommonResult byUrl(){
    return new CommonResult(200,"按url限流测试",new Payment(2020L,"serial002"));
}
````
* 2、新增流控规则
````
资源名: /rateLimit/byUrl
阈值类型: QPS 单机阈值: 1

表示1秒钟内查询次数大于1，就跑到我们自定义的兜底方法
````
## 3、上面兜底方案面临的问题
````
1、系统默认的，没有体现我们自己的业务要求
2、依照现有条件，我们自定义的处理方法又和业务代码耦合在一块，不直观
3、每个业务方法都添加一个兜底的，那代码膨胀加剧
4、全局统一的处理方法没有体现
````
## 4、客户自定义限流处理逻辑
* 1、CustomerBlockHandler 类用于自定义限流处理逻辑
````
public class CustomerBlockHandler {
    public static CommonResult handlerException1(BlockException exception){
        return new CommonResult(4444,"按用户自定义,global handlerException.......1");
    }
    public static CommonResult handlerException2(BlockException exception){
        return new CommonResult(4444,"按用户自定义,global handlerException.......2");
    }
}
````
* 2、controller新增
````
@GetMapping("/rateLimit/customerBlockHandler")
@SentinelResource(value = "customerBlockHandler",
                blockHandlerClass = CustomerBlockHandler.class,
                blockHandler = "handlerException2")
public CommonResult customerBlockHandler(){
    return new CommonResult(200,"按用户自定义",new Payment(2020L,"serial003"));
}

说明: 找CustomerBlockHandler类里的handleException2方法进行兜底处理
注意: URL地址流控优先级大于资源名称流控
````
## 5、更多注解属性说明
````
1、value：资源名称，必需项（不能为空）
2、entryType：entry类型，可选项（默认为 EntryType.OUT）
3、fallback：fallback函数名称，可选项，用于在抛出异常的时候提供 fallback处理逻辑。fallback函数可以针对所有类型的异常（除了exceptionsToIgnore里面排除掉的异常类型）进行处理。fallback函数签名和位置要求： 返回值类型必须与原函数返回值类型一致；方法参数列表需要和原函数一致，或者可以额外多一个 Throwable类型的参数用于接收对应的异常。
4、fallback函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 fallbackClass为对应的类的 Class 对象，注意对应的函数必需为 static函数，否则无法解析。 defaultFallback（since 1.6.0）：默认的 fallback函数名称，可选项，通常用于通用的 fallback逻辑（即可以用于很多服务或方法）。默认 fallback函数可以针对所有类型的异常（除了exceptionsToIgnore里面排除掉的异常类型）进行处理。若同时配置了 fallback和 defaultFallback，则只有 fallback会生效。defaultFallback函数签名要求：返回值类型必须与原函数返回值类型一致；
5、方法参数列表需要为空，或者可以额外多一个 Throwable 类型的参数用于接收对应的异常。
6、defaultFallback函数默认需要和原方法在同一个类中。若希望使用其他类的函数，则可以指定 fallbackClass为对应的类的 Class 对象，注意对应的函数必需为 static 函数，否则无法解析。
7、exceptionsToIgnore（since 1.6.0）：用于指定哪些异常被排除掉，不会计入异常统计中，也不会进入 fallback 逻辑中，而是会原样抛出。
````
# 九、服务熔断功能
## 1、Ribbon——提供者9003/9004
````
启动nacos和sentinel，新建cloudalibaba-provider-payment9003/9004
````
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

    <artifactId>cloudalibaba-provider-payment9003</artifactId>
    <dependencies>
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

        <!-- 日常通用jar包 -->
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
  port: 9003
spring:
  application:
    name: nacos-payment-provider
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
management:
  endpoints:
    web:
      exposure:
        include: '*'
````
* 3、主启动
````
@SpringBootApplication
@EnableDiscoveryClient
public class PaymentMain9003 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain9003.class, args);
    }
}
````
* 4、controller层
````
@RestController
public class PaymentController {
    @Value("${server.port}")
    private String serverPort;

    //模拟sql查询
    public static HashMap<Long, Payment> hashMap = new HashMap<>();
    static {
        hashMap.put(1L, new Payment(1L, "xcxcxcxcxcxcxcxcxcxcxcxcxc11111111"));
        hashMap.put(2L, new Payment(2L, "xcxcxcxcggggggggg2222222222222222"));
        hashMap.put(3L, new Payment(3L, "xcxcxcxccxxcxcfafdgdgdsgdsgds33333"));
    }


    @GetMapping("/payment/get/{id}")
    public CommonResult paymentSql(@PathVariable("id")Long id){
        Payment payment = hashMap.get(id);
        CommonResult result = new CommonResult(200, "from mysql, server port : " + serverPort + " ,查询成功", payment);
        return result;
    }
}
````
## 2、Ribbon——消费者84
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

    <artifactId>cloudalibaba-consumer-nacos-order84</artifactId>

    <dependencies>
        <!--SpringCloud openfeign -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
        <!--SpringCloud ailibaba nacos -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!--SpringCloud ailibaba sentinel -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
        </dependency>
        <!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
        <dependency>
            <groupId>com.jch.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>${project.version}</version>
        </dependency>
        <!-- SpringBoot整合Web组件 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--日常通用jar包配置-->
        <!--        <dependency>-->
        <!--            <groupId>org.springframework.boot</groupId>-->
        <!--            <artifactId>spring-boot-devtools</artifactId>-->
        <!--            <scope>runtime</scope>-->
        <!--            <optional>true</optional>-->
        <!--        </dependency>-->
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
    </dependencies>

</project>
````
* 2、application.yml
````
server:
  port: 84

spring:
  application:
    name: nacos-order-consumer
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
    sentinel:
      transport:
        #配置Sentinel dashboard地址
        dashboard: localhost:8888
        #默认8719端口，假如被占用会自动从8719开始依次+1扫描,直至找到未被占用的端口
        port: 8719

#消费者将要去访问的微服务名称(注册成功进nacos的微服务提供者)
service-url:
  nacos-user-service: http://nacos-payment-provider
````
* 3、主启动类 
````
@EnableDiscoveryClient
@SpringBootApplication
public class OrderMain84 {
    public static void main(String[] args) {
        SpringApplication.run(OrderMain84.class,args);
    }
}
````
* 4、配置类
````
@Configuration
public class ApplicationContextConfig {
    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
````
## 3、消费者@SentinelResource的各种配置
* 1、无配置
````
@RestController
@Slf4j
public class CircleBreakerController {
    private static final String PAYMENT_URL="http://nacos-payment-provider";
 
    @Resource
    private RestTemplate restTemplate;
 
    @GetMapping("/consumer/fallback/{id}")
    @SentinelResource(value = "fallback")//没有配置
    public CommonResult fallback(@PathVariable("id")Long id){
        if(id == 4){
            throw new IllegalArgumentException("IllegalArgumentException...非法参数异常");
        }else if(id > 4){
            throw new NullPointerException("NullPointerException...空指针异常");
        }
        return restTemplate.getForObject(PAYMENT_URL + "/payment/get/" + id, CommonResult.class);
    }
}
````
* 2、只配置fallback
````
@RestController
@Slf4j
public class CircleBreakerController {
    private static final String PAYMENT_URL="http://nacos-payment-provider";

    @Resource
    private RestTemplate restTemplate;

    @GetMapping("/consumer/fallback/{id}")
    @SentinelResource(value = "fallback", fallback = "handleFallback") //fallback只处理业务异常
    public CommonResult fallback(@PathVariable("id")Long id){
        if(id == 4){
            throw new IllegalArgumentException("IllegalArgumentException...非法参数异常");
        }else if(id > 4){
            throw new NullPointerException("NullPointerException...空指针异常");
        }
        return restTemplate.getForObject(PAYMENT_URL + "/payment/get/" + id, CommonResult.class);
    }

    //兜底方法
    public CommonResult handleFallback(@PathVariable("id")Long id, Throwable e){
        Payment payment = new Payment(id, null);
        return new CommonResult(444, "---兜底异常handlerFallback,exception内容："+e.getMessage(),payment);
    }
}
````
* 3、只配置blockHandler
````
@RestController
@Slf4j
public class CircleBreakerController {
    private static final String PAYMENT_URL="http://nacos-payment-provider";

    @Resource
    private RestTemplate restTemplate;

    @GetMapping("/consumer/fallback/{id}")
    @SentinelResource(value = "fallback",blockHandler = "blockHandler") //blockHandler 只负责sentinel控制台配置违规
    public CommonResult fallback(@PathVariable("id")Long id){
        if(id == 4){
            throw new IllegalArgumentException("IllegalArgumentException...非法参数异常");
        }else if(id > 4){
            throw new NullPointerException("NullPointerException...空指针异常");
        }
        return restTemplate.getForObject(PAYMENT_URL + "/payment/get/" + id, CommonResult.class);
    }

    public CommonResult blockHandler(@PathVariable("id")Long id, BlockException e){
        Payment payment = new Payment(id, "null");
        return new CommonResult(445, "blockHandler-sentinel限流,blockException:"+e.getMessage(),payment);
    }
}
````
````
新增服务熔断规则

资源名: fallback
降级策略: 异常数
异常数: 2 
熔断时长: 20 最小请求数: 5
统计时长: 1000
````
* 4、fallback和blockHandler都配置
````
@RestController
@Slf4j
public class CircleBreakerController {
    private static final String PAYMENT_URL="http://nacos-payment-provider";

    @Resource
    private RestTemplate restTemplate;

    @GetMapping("/consumer/fallback/{id}")
    @SentinelResource(value = "fallback", fallback = "handleFallback", blockHandler = "blockHandler")
                                //fallback只处理业务异常
                                //blockHandler 只负责sentinel控制台配置违规
    public CommonResult fallback(@PathVariable("id")Long id){
        if(id == 4){
            throw new IllegalArgumentException("IllegalArgumentException...非法参数异常");
        }else if(id > 4){
            throw new NullPointerException("NullPointerException...空指针异常");
        }
        return restTemplate.getForObject(PAYMENT_URL + "/payment/get/" + id, CommonResult.class);
    }

    public CommonResult handleFallback(@PathVariable("id")Long id, Throwable e){
        Payment payment = new Payment(id, null);
        return new CommonResult(444, "---兜底异常handlerFallback,exception内容："+e.getMessage(),payment);
    }

    public CommonResult blockHandler(@PathVariable("id")Long id, BlockException e){
        Payment payment = new Payment(id, "null");
        return new CommonResult(445, "blockHandler-sentinel限流,blockException:"+e.getMessage(),payment);
    }
}
````
````
限流之前进入 handleFallback
限流之后只会进入 blockHandler
````
* 5、忽略异常
````
@RestController
@Slf4j
public class CircleBreakerController {
    private static final String PAYMENT_URL="http://nacos-payment-provider";
 
    @Resource
    private RestTemplate restTemplate;
 
    @GetMapping("/consumer/fallback/{id}")
    @SentinelResource(value = "fallback",blockHandler = "blockHandler",fallback = "handleFallback",
            exceptionsToIgnore = {IllegalArgumentException.class})
    //exceptionsToIgnore有这个异常不走兜底方法
    public CommonResult fallback(@PathVariable("id")Long id){
        if(id == 4){
            throw new IllegalArgumentException("IllegalArgumentException...非法参数异常");
        }else if(id > 4){
            throw new NullPointerException("NullPointerException...空指针异常");
        }
        return restTemplate.getForObject(PAYMENT_URL + "/payment/get/" + id, CommonResult.class);
    }
 
    //兜底方法
    public CommonResult handleFallback(@PathVariable("id")Long id, Throwable e){
        Payment payment = new Payment(id, null);
        return new CommonResult(444, "---兜底异常handlerFallback,exception内容："+e.getMessage(),payment);
    }
 
    public CommonResult blockHandler(@PathVariable("id")Long id, BlockException e){
        Payment payment = new Payment(id, "null");
        return new CommonResult(445, "blockHandler-sentinel限流,blockException:"+e.getMessage(),payment);
    }
}
````
## 4、全局降级
* 1、添加open-feign依赖
````
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
````
* 2、yml 追加如下配置
````
# 激活Sentinel对Feign的支持
feign:
  sentinel:
    enabled: true
````
* 3、主启动类添加注解: @EnableFeignClients 激活open-feign
````
@EnableDiscoveryClient
@SpringBootApplication
@EnableFeignClients
public class OrderMain84 {
    public static void main(String[] args) {
        SpringApplication.run(OrderMain84.class,args);
    }
}
````
* 4、service
````
@FeignClient(value = "nacos-payment-provider", fallback = PaymentFallbackService.class)
public interface PaymentService {
    @GetMapping(value = "/payment/get/{id}")
    public CommonResult paymentSql(@PathVariable("id")Long id);
    //方法名也要和provider-payment9003/9004里的一样
}
````
````
@Component
public class PaymentFallbackService implements PaymentService {
    @Override
    public CommonResult paymentSql(Long id) {
        return new CommonResult(44444,"服务降级返回,---PaymentFallbackService");
    }
}
````
* 5、controllert
````
@GetMapping(value = "/consumer/paymentSQL/{id}")
public CommonResult paymentSQL(@PathVariable Long id) {
    return paymentService.paymentSql(id);
}
````
* 6、测试
````
此时关闭9003微服务提供者, 84消费者会触发服务降级
````
# 十、持久化-以流控规则为例
````
目前的sentinel 当重启以后，数据都会丢失，和 nacos 类似原理。需要持久化。它可以被持久化到 nacos 的数据库中
````
* 1、pom文件
````
<dependency>
    <groupId>com.alibaba.csp</groupId>
    <artifactId>sentinel-datasource-nacos</artifactId>
</dependency>
````
* 2、application.yml
````
server:
  port: 8401
spring:
  application:
    name: cloudalibaba-sentinel-service
  cloud:
    nacos:
      discovery:
        # 服务注册中心 # sentinel注册进nacos
        server-addr: localhost:8848
    sentinel:
      transport:
        # 配置 Sentinel Dashboard 的地址
        dashboard: localhost:8888
        # 默认8719 ，如果端口被占用，端口号会自动 +1，直到找到未被占用的端口，提供给 sentinel 的监控端口
        port: 8719
      datasource:
        ds1:
          nacos:
            server-addr: localhost:8848
            dataId: ${spring.application.name}
            groupId: DEFAULT_GROUP
            data-type: json
            rule-type: flow
management:
  endpoints:
    web:
      exposure:
        include: '*'
````
* 3、nacos上新建配置文件, 
````
Data ID: cloudalibaba-sentinel-service
配置格式: JSON
配置内容: 
[
    {
        "resource": "fallback",
        "limitApp": "default",
        "grade": 1,
        "count": 1,
        "strategy": 0,
        "controlBehavior": 0,
        "clusterMode": false
    }
]
````
````
resource：资源名称
limitApp：来源应用
grade：阈值类型，0表示线程数，1表示QPS，
count：单机阈值，
strategy：流控模式，0表示直接，1表示关联，2表示链路
controlBehavior:流控效果，0表示快速失败，1表示Warm Up,2表示排队等待；
cIusterM0de是否集群
````
* 4、新建流控规则
````
资源名: /rateLimit/byUrl
阈值类型: QPS 单机阈值: 1
````
* 5、测试
````
快速访问 http://localhost:8401/rateLimit/byUrl, 流控规则生效

关闭8401 sentinel微服务模块, 发现流控规则消失

重启8401 sentinel微服务模块, 发现流控规则出现
````