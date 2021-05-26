> [官网地址](https://cloud.spring.io/spring-cloud-static/spring-cloud-config/2.2.1.RELEASE/reference/html/)
* cloud-config-center3344
* cloud-config-client3366

# Config
> 配置服务器为不同微服务应用提供一个中心化的外部配置
```pom
## server
server:
  port: 3344

spring:
  application:
    name: cloud-config-center
  profiles:
    active: git
  cloud:
    config:
      server:
        git:
          uri: https://github.com/nanami-yu/spring-cloud-config.git
          username:
          password:
          search-paths:
            - spring-cloud-config
          default-label: main
eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/


## client bootstrap.yml
server:
  port: 3366

spring:
  application:
    name: config-client
  cloud:
    config:
      label: main
      name: config
      profile: dev
      uri: http://localhost:3344

eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka/,http://eureka7002.com:7002/eureka/

```

## 动态加载
```java
# 图形化监控
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>


# 监控端口
management:
  endpoints:
    web:
      exposure:
        include: "*"

# 业务类上方添加注解 @RefreshScope

# 发送post请求 提示刷新
curl -X POST "http://localhost:3366/actuator/refresh"

# 避免客户端重启
```
## 遗留问题
* 多个微服务 需要使用消息总线