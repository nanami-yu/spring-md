> [官网地址](github.com/Netflix/ribbon)
> * order80
> * eureka7001
> * eureka7002
> * payment8001
> * payment8002
## 负载均衡
> 将用户请求平摊到多个服务上 客户端nginx 本地负载均衡ribbon
```pom
        #已经包含ribbon 无需再引入
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
            <version>2.2.2.RELEASE</version>
        </dependency>
```
```java
        // restTemplate建议使用entity查询
        ResponseEntity<CommonResult> common =  restTemplate.getForEntity(PAYMENT_URL + "/payment/get/" + id,CommonResult.class);

        if(common.getStatusCode().is2xxSuccessful()){
            return common.getBody();
        }else{
            return new CommonResult<>(444,"操作失败");
        }
```