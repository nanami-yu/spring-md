> [官网地址](https://github.com/OpenFeign/feign)
> * cloud-consumer-feign-order80
> * eureka7001
> * eureka7002
> * payment8001
> * payment8002
## OpenFeign是什么
> 是一个声明式的Web服务客户端，只需要创建一个接口并添加注解即可

## Feign能干什么
> 简化ribbon 直接得到服务提供方接口的方法 核心注解@FeignClient
* pom
```xml
        <!--openfeign-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
        </dependency>
```
* yml
```yml
server:
  port: 80

spring:
  application:
    name: cloud-order-service

eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/
```
* java <span style="color:red;">(注意：service要与服务提供者controller相同)</span>
```java
// main
@SpringBootApplication
@EnableFeignClients
public class OrderFeign80 {

    public static void main(String[] args) {
        SpringApplication.run(OrderFeign80.class,args);
    }

}

// service
@Component
@FeignClient(value = "CLOUD-PROVIDER-SERVICE")
public interface PaymentFeignService {

    @GetMapping(value = "/payment/get/{id}")
    CommonResult getPaymentById(@PathVariable("id") int id);
}

// controller
@RestController
@Slf4j
public class OrderFeignController {

    @Resource
    private PaymentFeignService paymentFeignService;

    @GetMapping(value = "/consumer/get/{id}")
    public CommonResult<Payment> getById(@PathVariable("id") int id){
        CommonResult<Payment> result = null;
        try{
            result = paymentFeignService.getPaymentById(id);
        }catch (Exception e){
            log.info(e.getMessage());
        }
        return result;
    }
}
```
## 超时控制
> 默认等待1秒
```java
// 提供者
@GetMapping(value = "/timeout")
public String timeout(){
    try{
        TimeUnit.SECONDS.sleep(3);
    }catch (Exception e){
        e.printStackTrace();
    }
    return serverPort;
}

// client
@GetMapping(value = "/timeout")
public String timeout(){
    String result = null;
    try{
        result = paymentFeignService.timeout();
    }catch (Exception e){
        log.info(e.getMessage());
    }
    return result;
}

// yml配置
ribbon:
  ReadTimeout: 5000
  ConnectTimeout: 5000
```

## 日志打印功能
* NONE    无
* BASIC   基础信息
* HEADERS 头信息
* FULL    详细
```java
@Configuration
public class FeignConfig {
    @Bean
    Logger.Level feignlv(){
        return Logger.Level.FULL;
    }
}

logging:
  level:
    # feign日志监控文件及级别
   com.atguigu.service.PaymentFeignService: debug

```