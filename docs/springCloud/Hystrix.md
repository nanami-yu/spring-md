> [官网地址](https://github.com/Netflix/Hystrix)
> * cloud-provider-hystrix-payment8001
> * cloud-consumer-feign-hystrix-order80
> * eureka7001
> * eureka7002
## Hystrix是什么
> 用于处理分布式系统的延迟和容错，能够保证在一个依赖出错的情况下，不会导致整体服务失败，避免级联故障
> "断路器"本身是一种开关装置，当某个单元发生故障之后通过监控，向调用方返回一个
> 符合预期、可处理的响应，而不是等待或抛出异常

## 重要概念
### 服务降级
> 当服务不可用时，提供一个备选响应

### 服务熔断
> 当服务最大访问时，调用服务降级返回信息

### 服务限流
> 高并发限制访问数量

## 依赖
```sh
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

## 高并发
> 高并发下需要有解决方案
* 1.调用的服务超时
* 2.调用的服务宕机
* 3.自身服务出现问题

## 服务降级
> 服务预期响应与备用响应 一般放在客户端 修改相关代码需要重启微服务
```java
    // 服务端配置方法fallback
    @HystrixCommand(fallbackMethod = "timeOutHandler",commandProperties = {
            @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "3000")
    })

    // 服务端main方法
    @EnableCircuitBreaker
```
> @HystrixCommand中设置的fallback方法 在程序遇到错误时会直接找到fallback方法

```java
// 客户端 pom
feign:
  hystrix:
    enabled: true # 开启feign对hystrix的支持

// 客户端 main
@EnableHystrix

//  客户端 controller
@GetMapping("/timeout/{id}")
@HystrixCommand(fallbackMethod = "timeOutHandler",commandProperties = {
        @HystrixProperty(name = "execution.isolation.thread.timeoutInMilliseconds",value = "1500")
})
String paymentInfo_timeout(@PathVariable("id") Integer id){
    return paymService.paymentInfo_timeout(id);
}

public String timeOutHandler(Integer id){
    return "服务超时";
}
```
### 问题
> 会导致代码膨胀 解决方法为设置一个通用的fallback</br>@HystrixCommand不指定方法 调用全局方法
```java
@DefaultProperties(defaultFallback = "global_fallback")

@GetMapping("/global/{id}")
@HystrixCommand
public String tryGlobalFallback(@PathVariable("id") Integer id){
    return paymService.paymentInfo_timeout(id);
}

public String global_fallback(){
    return "全局fallback";
}
```
> 业务逻辑混乱 代码耦合度高 </br> 通过feign添加统一的服务降级类
```java
// feign接口类
@FeignClient(value = "cloud-provider-hystrix-payment",fallback = PaymServiceImpl.class)

// 实现类
@Component
public class PaymServiceImpl implements PaymService{

    @Override
    public String paymentInfo_OK(Integer id) {
        return "----feign paymentInfo_OK fallback----";
    }

    @Override
    public String paymentInfo_timeout(Integer id) {
        return "----feign paymentInfo_timeout fallback----";
    }
}
```
> 通过实现可以覆盖controller中的全局 但是无法覆盖指定的方法

## 服务熔断
>