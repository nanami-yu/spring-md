# Nacos
> Naming和Configuration,为注册中心+配置中心的组合 = Eureka+config+Bus

## 官网
> nacos.io/zh-cn/

### 下载
> 本地需要有java及maven环境  [nacos下载](https://github.com/alibaba/nacos/releases/tag/1.1.4)
```shell
1.进入到解压文件夹 执行 startup.cmd
2.访问http://localhost:8848/nacos
  默认账号密码nacos/nacos
```

## 注册中心
> * cloudalibaba-provider-payment9001
> * cloudalibaba-provider-payment9002
> * cloudalibaba-consumer-order83

* pom
```sh
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
    </dependency>
</dependencies>
```

* yml
```yml
server:
  port: 9002
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
```

* java
```java
// 服务提供者
// 启动类
@SpringBootApplication
@EnableDiscoveryClient
public class Provider9002 {
    public static void main(String[] args) {
        SpringApplication.run(Provider9002.class, args);
    }
}
// controller
@RestController
public class PaymentController {

    @Value("${server.port}")
    private String port;

    @GetMapping(value="/payment/nacos/{id}")
    public String getPayment(@PathVariable("id")String id){
        return "nacos,port:" + port + "\t id:" +id;
    }
}
// 服务消费者
// 启动类
@SpringBootApplication
@EnableDiscoveryClient
public class Order83 {
    public static void main(String[] args) {
        SpringApplication.run(Order83.class,args);
    }
}
// controller
@RestController
public class OrderController {

    @Resource
    private RestTemplate restTemplate;

    @Value("${service-url.nacos-user-service}")
    private String serverURL;

    @GetMapping(value = "/consumer/payment/nacos/{id}")
    public String paymentInfo(@PathVariable("id")String id){
        return restTemplate.getForObject(serverURL+"/payment/nacos/"+id,String.class);
    }
}
// config
@Configuration
public class ApplicationContextConfig {

    @Bean
    @LoadBalanced
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }
}
```

## 测试
> * http://localhost:83/consumer/payment/nacos/13
> * http://localhost:8848/nacos/#/serviceManagement?dataId=&group=&appName=&namespace=

## CAP
> Nacos支持AP、CP切换。CP模式主要用于k8s及DNS

## 配置中心
* cloudalibaba-config-nacos-client3377
* pom
```pom
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
        </dependency>
        <!-- 配置中心 -->
        <dependency>
            <groupId>com.alibaba.cloud</groupId>
            <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
        </dependency>
    </dependencies>
```

> springBoot配置文件加载顺序为boostrap>application,所以需要创建两个配置文件
* bootstrap
```yml
server:
  port: 3377
spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
      config:
        server-addr: localhost:8848
        file-extension: yaml
#${spring.application.name}-${spring.profile.active}-${spring.cloud.nacos.config.file-extension}
```

* yml
```yml
spring:
  profiles:
    active: dev
```

* java
```java
// 启动类
@SpringBootApplication
@EnableDiscoveryClient
public class Config3377 {
    public static void main(String[] args) {
        SpringApplication.run(Config3377.class,args);
    }
}
// controller
@RestController
@RefreshScope // 动态刷新
public class ConfigController {

    @Value("${config.info}")
    private String info;

    @GetMapping("/config/info")
    public String getInfo(){
        return info;
    }
}
```
### dataId
>  (nacos-config-client)-(dev).(yml)

### 新增配置
#### 配置地址
> http://localhost:8848/nacos/#/newconfig?serverId=center&namespace=&edasAppName=&edasAppId=&searchDataId=&searchGroup=

#### 配置内容
```yml
config:
    info: 1
```

#### 测试
> http://localhost:3377/config/info

## 分类管理
> 为了应对复杂的环境配置,使用命名空间+Group+Data ID区分，默认的命名空间为public,默认分组为DEFAULT_GROUP,默认集群为DEFAULT

### DataID方案
> 指定spring.profile.active和配置文件的DataID来使不用环境下读取不同的配置

### Group方案
> 指定group来使不用环境下读取不同的配置
* bootstrap
```yml
spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
      config:
        server-addr: localhost:8848
        file-extension: yaml
        group: nanami # 指定group
```

### Namespace方案
> 指定Namespace来使不用环境下读取不同的配置
* bootstrap
```yml
spring:
  application:
    name: nacos-config-client
  cloud:
    nacos:
      discovery:
        server-addr: localhost:8848
      config:
        server-addr: localhost:8848
        file-extension: yaml
        group: nanami
        namespace: 78b05076-a3bf-4a62-99f2-a47e51142f71
```

## 集群与持久化
> https://nacos.io/zh-cn/docs/cluster-mode-quick-start.html
>
> https://zhuanlan.zhihu.com/p/150400342
>
> https://www.jianshu.com/p/55a4201dc78d
>
> http://10.4.7.131/nacos (集群访问地址)

> 具体配置文件查看static/nacos