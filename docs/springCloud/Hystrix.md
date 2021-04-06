> [官网地址](https://github.com/Netflix/Hystrix)
> * cloud-provider-hystrix-payment8001
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

## 项目
```sh
// 依赖
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```