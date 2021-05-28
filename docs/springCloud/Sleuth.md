## 为什么会使用
> 微服务架构会导致后端接口多段链路调用,需要知道每个请求的链路。
> [官网](https://github.com/spring-cloud/spring-cloud-sleuth)、
> Sleuth提供了一套完整的链路调用查看功能

## 搭建链路监控
### zipkin
```sh
docker pull openzipkin/zipkin
firewall-cmd --zone=public --add-port=9411/tcp --permanent
firewall-cmd --reload
docker run --name zipkin -d -p 9411:9411 openzipkin/zipkin
http://167.179.88.93:9411/zipkin/
```
### 链路
> 每个请求记录父节点与当前节点,汇总到zipkin上

### 调用
> 使用的项目
> * cloud-consumer-order80
> * cloud-provider-payment8001
```yml
# 依赖
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>

# 配置文件
spring:
  application:
    name: cloud-provider-service
  zipkin:
    base-url: http://167.179.88.93:9411
  sleuth:
    sampler:
      probability: 1 # 采样率0~1 1表示全部采集
```
```java
    // 被调用接口
    @GetMapping(value = "/payment/zipkin")
    public String zipkin(){
        return "zipkin";
    }
    // 调用接口
    @GetMapping("/consumer/payment/zipkin")
    public String paymentZipkin(){
        String res =  restTemplate.getForObject(PAYMENT_URL + "/payment/zipkin" ,String.class);
        return res;
    }
```
> 访问 http://localhost/consumer/payment/zipkin 查看链路