> [官网地址](github.com/Netflix/ribbon)
> * order80
> * eureka7001
> * eureka7002
> * payment8001
> * payment8002
## 负载均衡
> 将用户请求平摊到多个服务上 客户端nginx 本地负载均衡ribbon
```java
        #已经包含ribbon 无需再引入
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
            <version>2.2.2.RELEASE</version>
        </dependency>

        // restTemplate建议使用entity查询
        ResponseEntity<CommonResult> common =  restTemplate.getForEntity(PAYMENT_URL + "/payment/get/" + id,CommonResult.class);

        if(common.getStatusCode().is2xxSuccessful()){
            return common.getBody();
        }else{
            return new CommonResult<>(444,"操作失败");
        }
```
* RoundRobinRule             轮询
* RandomRule                 随机
* RetryRule                  先按照轮询 失败会重试
* WeightedResponseTimeRule   轮巡扩展 响应速度越快 越容易被选中
* BestAvailableRule          过滤到由于访问故障的服务，在可用的服务中选择并发最小的
* AvailabilityFilteringRUle  在可用的服务中选择并发最小的
* ZoneAvoidanceRule          默认 根据性能可用性选择

## 配置
> Robin配置类不能放在可以被ComponentScan扫描到的位置
```java
@Configuration
public class MySelfRule {
    @Bean
    public IRule myRule(){
        return new RandomRule();
    }
}

// 主启动类添加
@RibbonClient(name = "CLOUD-PAYMENT-SERVICE",configuration = MySelfRule.class)
```

## 源码
> 轮询依据主要是获取所有服务 根据请求次数 获取角标
> 原理 + JUC(CAS + 自旋锁)
```java
package com.atguigu.springcloud.lb;

import org.springframework.cloud.client.ServiceInstance;
import org.springframework.stereotype.Component;

import java.util.List;
import java.util.concurrent.atomic.AtomicInteger;

@Component
public class MyLB implements LoadBalancer{

    private AtomicInteger atomicInteger = new AtomicInteger(0);

    public final int getAndIncrement(){
        int current;
        int next;
        do {
            current = this.atomicInteger.get();
            next = current >= Integer.MAX_VALUE ? 0 : current + 1;
        } while(!this.atomicInteger.compareAndSet(current,next));
        return next;
    }

    @Override
    public ServiceInstance instances(List<ServiceInstance> serviceInstanceList) {
        int index = getAndIncrement() % serviceInstanceList.size();
        return serviceInstanceList.get(index);
    }

}
```