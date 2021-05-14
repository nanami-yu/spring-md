> [官网地址](https://spring.io/projects/spring-cloud-gateway)
* cloud-gateway-gateway9527
* cloud-provider-payment8001

> 提供一种简单而有效的方式对API进行路由,以及熔断、限流、重试等功能
> gateway是基于异步非阻塞模型开发，性能强
* 动态路由
* 集成Hystrix
* 支持断言，过滤器

## 三大核心概念
### 1.路由
> 由ID，目标URL，一系列的单元和过滤器组成

### 2.断言
> 可以匹配HTTP请求中的所有内容,如果请求与断言相匹配则进行路由

### 3.过滤
> 使用过滤器,可以在请求被路由前、后对请求进行修改

## 配置
### 核心依赖
```pom
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
    </dependencies>
```
### yml 配置方式
```yml
spring:
  application:
    name: cloud-gateway-service
  cloud:
    gateway:
      routes:
        - id: payment_routh1
          uri: http://localhost:8001
          predicates:
            - Path=/payment/get/**


        - id: payment_routh2
          uri: http://localhost:8001
          predicates:
            - Path=/payment/lb/**
```
### RouteLocator 配置方式
```java
@Configuration
public class GatewayConfig {
    @Bean
    public RouteLocator routes(RouteLocatorBuilder routeLocatorBuilder){
        RouteLocatorBuilder.Builder routes = routeLocatorBuilder.routes();
        routes.route("router1",r -> r.path("/guonei").uri("http://news.baidu.com/guonei")).build();
        return routes.build();
    }
}
```
### 动态路由 负载均衡
```yml
spring:
  application:
    name: cloud-gateway-service
  cloud:
    gateway:
      discovery:
        locator:
          enabled: true # 开启从注册中心动态创建路由功能,利用微服务名进行路由
      routes:
        - id: payment_routh1
          # uri: http://localhost:8001
          uri: lb://CLOUD-PROVIDER-SERVICE
          predicates:
            - Path=/payment/get/**


        - id: payment_routh2
          # uri: http://localhost:8001
          uri: lb://CLOUD-PROVIDER-SERVICE
          predicates:
            - Path=/payment/lb/**
```
### Predicate(断言)
```yml
- Path=/payment/get/**
- After=2021-05-14T16:11:42.224+08:00[Asia/Shanghai]
- Before=2021-05-14T16:11:42.224+08:00[Asia/Shanghai]
- Between=2021-05-14T16:11:42.224+08:00[Asia/Shanghai],2021-05-14T16:11:42.224+08:00[Asia/Shanghai]
- Cookie=username,fy # --coolie "username=fy"
- Header=X-Request-Id,\d+ # -H "=X-Request-Id:1"
- Host=**.somehost.org,**.anotherhost.org # -H "=Host:ce.somehost.org"
- Mehtod=GET
- Query=username,\d+ # localhost:9527/payment/lb?username=1
```
### filter(过滤器)
> 生命周期 pre/post

> 种类 GatewayFilter/GlobalFilter
#### 自定义过滤器
```java
@Component
public class MyLogGateWayFilter implements GlobalFilter, Ordered {

    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        System.out.println("*******************************过滤器:" + new Date());
        String uname = exchange.getRequest().getQueryParams().getFirst("uname");
        if(uname == null){
            System.out.println("非法");
            exchange.getResponse().setStatusCode(HttpStatus.NOT_ACCEPTABLE);
            return exchange.getResponse().setComplete();
        }
        return chain.filter(exchange);
    }

    @Override
    public int getOrder() {
        return 0;
    }
}

```