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