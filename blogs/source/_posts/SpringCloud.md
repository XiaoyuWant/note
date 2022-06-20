---
title: SpringCloud 笔记
date: 2021-06-12
---


# SpringCloud 笔记

## 1. 服务注册中心

### Eureka

现在已经不再更新。

分为 client 和 server，需要在 `application.yml`中手动配置地址才能互相发现。 

### Consul

同上

### Zookeeper

pom.xml配置

```xml
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
</dependency>
```

application.yml 配置，加上zookeeper服务端的地址

```yaml
spring:
  application:
    name: cloud-provider-payment
  cloud:
    zookeeper:
      connect-string: 121.5.57.114:2181
```

注册后是临时节点，保证了CP

在webContextConfig配置 restTemplate

```java
@Bean
@LoadBalanced
public RestTemplate getRestTemplate(){
    return new RestTemplate();
}
```

主启动类开启发现注解

```java
@SpringBootApplication
@EnableDiscoveryClient
public class ConsumerZkMain80 {
    public static void main(String[] args) {
        SpringApplication.run(ConsumerZkMain80.class,args);
    }
}
```

Controller中使用注册在zookeeper的name访问服务，如

```java
@RestController
@Slf4j
public class OrderController {
    public static final String PAYMENT_URL="http://CLOUD-PAYMENT-SERVICE";

    @Resource
    private RestTemplate restTemplate;


    @GetMapping("/consumer/payment/zk")
    public String paymentInfo(){
        String result = restTemplate.getForObject(PAYMENT_URL + "/payment/zk", String.class);
        return result;
    }
}
```

### Nacos

// 见后面

## 2. 服务调用

### Ribbon

### OpenFeign

同样需要在`application.yml`注册zookeeper

```yaml
spring:
  application:
    name: cloud-consumer-service
  cloud:
    zookeeper:
      connect-string: 121.5.57.114:2181
```

主启动类。

```java
@SpringBootApplication
@EnableFeignClients // 注册Feign
@EnableDiscoveryClient // 注册zookeeper
public class ConsumerMainFeign80 {
    public static void main(String[] args) {
        SpringApplication.run(ConsumerMainFeign80.class,args);
    }
}
```

写接口服务 `service`注意服务名称与zookeepe注册的大小写必须一致，不然找不到服务

```java
@Component
@FeignClient(value = "cloud-payment-service") // 注册Feign要调用的服务名称
public interface PaymentFeignService {

    @GetMapping(value = "/payment/zk")
    public String paymentZk();
}
```

主`controller`

```java
@RestController
@Slf4j
public class OrderFeignController {
    @Resource
    private PaymentFeignService paymentFeignService;

    @GetMapping("/consumer/payment/zk")
    public String getPaymentById(){
        return paymentFeignService.paymentZk();
    }
}
```

超时控制 `application.yml`

```
ribbon:
	ReadTimeout: 5000
	ConnectTimeout: 5000
```

日志打印 `confg/FeignConfig` 以及`application.yml`

```java
@Configuration
public class FeignConfig {
    @Bean
    Logger.Level feignLoggerLevel(){
        return Logger.Level.FULL;
    }
}
```

```yaml
logging:
  level:
    com.xiaoyu.cloud.service.PaymentFeignService: debug
```

## 3. 服务降级

### Hystrix  [deprecated]

服务间链式调用，即 扇出。一个服务失效不会导致整体服务失败，避免级联故障，提高分布式系统的弹性。

- 降级，Fallback
- 熔断，在错误达到一定情况下进行熔断，符合条件后继续开启
- 限流
- 实时监控

### Sentinel

// 见后面

## 4. 服务网关

### Zull 

路由网关。阻塞型IO，是个Tomcat容器。

### Gateway

基于 `Springboot2` & `WebFlux` & `Reactor `，底层使用了`Netty`。路由转发+过滤器链

客户端向Spring Cloud Gateway发送请求，Gateway Handler Mapping中找到与请求匹配的路由，将其发送到Gateway Web Handler。Handler通过指定的过来不起链 将请求发送到实际的服务执行业务逻辑。

过滤器之间有pre和post的执行逻辑。如参数校验、流量监控、日志输出、协议转换；post可以相应内容、响应头修改、日志输出、流量监控。

主启动类，配置zookeeper发现。`pom.xml`需要移除 `spring-boot-web-starter & actuator`

`application.yml` 。

```yaml
spring:
  application:
    name: cloud-gateway-service
  cloud:
    zookeeper:
      connect-string: 121.5.57.114:2181
    gateway:
      discovery:
        locator:
          enabled: true # 开启从注册中心创建路由
      routes:
        - id: payment_routeh
#          uri: http://localhost:8004
          uri: lb://cloud-payment-service
          predicates:
            - Path=/payment/zk/**
        - id: payment_route2
#          uri: http://localhost:8004
          uri: lb://cloud-payment-service
          predicates:
            - Path=/payment/lb/**
```

- 使用loadbalancer+微服务名称作为uri地址，实现动态路由及负载均衡。

- predicates: 配置断言条件，如`Path Cookie Header Method `等

- filters: 在`application.yml`可以配置但是不够明确，一般采用手写自定义全局过滤器 `MyLogGatewayFilter`

```java
@Component
public class MyLogGatewayFilter implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain){
        String uname = exchange.getRequest().getQueryParams().getFirst("uname");
        if(uname==null){
            System.out.println("非法用户");
            exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);
            return exchange.getResponse().setComplete();
        }
        return chain.filter(exchange);
    }
    @Override
    public int getOrder(){
        return 0;
    }
}
```



## 5. 服务配置 & 服务总线

### Bus & SpringCloud Config

- SpringCloud Config：集中管理配置文件， 用git进行管理，配置信息以REST接口形式暴露；修改git的配置并添加@RefreshScope，需要运维发送POST请求，/actuator/refresh 才能真正刷新配置 

- Bus：配合RabbitMQ，消息总线，使用轻量的消息代理构建一个公用的消息主题，主题的消息被所有的实例监听和消费，方便地广播需要让其他连接在该主题的实力都知道的消息。
  - 可以发送到某个服务上，蔓延到全部；也可以发到ConfigServer，这样更好，体现了微服务的单一功能性，并且方便迁移。

RabbitMQ在应用的配置信息: 有个默认的SpringCloudBus主题，接受的就是bus消息。只需要引入库，不需要手写controller。

`	spring-cloud-starter-bus-amqp`

```yaml
rabbitmq:
	host: 121.5.57.114
	port: 5672
	username: guest
	password: guest
```

刷新POST请求广播：`http://loaclhost:3344/actuator/bus-refresh `

定点通知：`http://loaclhost:3344/actuator/bus-refresh/{config-client:3355}`

### SpringCloud Stream：

构建消息驱动微服务的框架。通过配置来Binding，Stream与binder对象负责与消息中间件的交互。通过Integration来连接消息代理中间件实现消息事件驱动。为消息中间件产品提供了个性化的自动化配置，引用了发布-订阅、消费组、分区的三个核心概念。`spring-cloud-starter-stream-rabbit`

- docker安装rabbitMq需要进入容器启动management

发送端：在service.impl中实现

```java
@EnableBinding(Source.class) // 定义消息的推送管道
public class MessageProvider implements IMessageProvider {

    @Resource
    private MessageChannel output;

    @Override
    public String send() {
        String serial = UUID.randomUUID().toString();
        output.send(MessageBuilder.withPayload(serial).build());
        System.out.println("******Serial:"+serial);
        return null;
    }
}
```

接收端：直接在controller实现

```java
@Component
@EnableBinding(Sink.class)
public class ReceiveMessageListenerController {
    @Value("${server.port}")
    private String serverPort;

    @StreamListener(Sink.INPUT)
    public void input(Message<String> message){
        System.out.println("消费者1-->接收消息"+message.getPayload()+"\t"+"port"+serverPort);
    }
}
```

但是会产生重复消费的问题，都接受到了相同的消息。需要使用Stream的分组解决。不同组可以重复消费，同一组会竞争，只有一个消费。

配置文件加入`group`可以制定分组。

### Sleuth 

分布式请求链路跟踪。安装zepkin。

## SpringCloud Alibaba

### Nacos

动态服务发现、配置管理、服务管理平台

#### 服务发现

引入 `spring-cloud-alibaba-dependencies` `spring-cloud-alibaba-nacos-discovery`

配置文件：

```yaml
server:
  port: 9001
spring:
  application:
    name: alibaba-nacos-provider
  cloud:
    nacos:
      discovery:
        server-addr: 121.5.57.114:8848
```

以及`group` `namespace`的设置，作为应用的配置隔离。

动态获取配置：首先读取`bootstrap.yaml `然后从nacos中读取`name-method.yaml`，这个配置可以修改nacos的配置来制定Mysql作为配置数据库。

并且可以同时启动多个Nacos服务，使用nginx反向代理，实现高可用的nacos集群。

### Sentinel

分布式系统的流量控制。

限流：
- QPS直接失败、线程数直接失败；
- 关联、预热、排队等待。

降级：
- RT平均响应时间：1s进入5个请求&超过平均响应时间，触发断路器；
- 异常比例：请求异常的比例超出的时候，就会触发断路器；
- 异常数：

热点Key
- 降级方法`@SentinelResource(value="name",blockHandler="blockHandler",fallback="fallbackHandler")`


服务熔断：结合OpenFeign的使用



### Seata
分布式事务微服务解决方案，依靠全局唯一事务ID
- TC 事务协调器
- TM 事务管理
- RM 资源管理器
