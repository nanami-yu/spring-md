## 依赖
```yml
        # 客户端依赖
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
            <version>2.2.1.RELEASE</version>
        </dependency>

        # 服务端依赖
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
            <version>2.2.2.RELEASE</version>
        </dependency>
```
* 服务端主启动类需要添加 @EnableEurekaServer
* 客户端主启动类需要添加 @EnableEurekaClient

## yml
```yml
# 客户端
eureka:
  instance:
    hostname: eureka7001.com # 实例名称
  client:
    # 不注册本身
    register-with-eureka: false
    # 是否要获取其他服务地址
    fetch-registry: false
    service-url:
      # 高可用相互注册 单机版自己注册自己
      defaultZone: http://eureka7002.com:7002/eureka/

# 服务端
eureka:
  client:
    register-with-eureka: true
    fetch-registry: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka,http://eureka7002.com:7002/eureka
  instance:
    instance-id: payment8001 # 实例名称
    prefer-ip-address: true # 显示ip地址
```

## actuator

* 主机名称修改
```yml
eureka:
  instance:
    instance-id: payment8001 # 实例名称
    prefer-ip-address: true # 显示ip地址
```
![actuator](/static/jpg/actuator.png)
>查看健康状态为/actuator/health

## Discovery
>服务发现 主启动类添加@EnableDiscoveryClient
```java
    @Resource
    private DiscoveryClient discoveryClient;


    @GetMapping(value = "/payment/discovery")
    public Object discovery(){
        // 查询所有服务
        List<String> services = discoveryClient.getServices();
        for(String element : services){
            log.info("****element" + element);
        }
        // 查询CLOUD-PROVIDER-SERVICE服务下每个实例信息
        List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PROVIDER-SERVICE");
        for(ServiceInstance instance : instances){
            log.info("****instance" + instance);
        }
        return this.discoveryClient;
    }
```
## 自我保护
![自我保护](/static/jpg/eureka自我保护.png)
>在一定时间内服务器没有收到心跳包 会开始保护机制90s内不会删除服务