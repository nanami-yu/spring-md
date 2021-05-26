> [官网地址](https://cloud.spring.io/spring-cloud-static/spring-cloud-config/2.2.1.RELEASE/reference/html/)
* cloud-config-center3344
* cloud-config-client3366

# Bus
> 为了实现动态配置更新,支持rabbitMQ和kafka
![bus1](../static/jpg/Bus.png)
![bus2](../static/jpg/Bus2.jpg)

## 消息总线
> 让微服务中所有实例共用一个消息主题，称之为消息总线

### mq相关讲解
[mq](https://www.bilibili.com/video/av55976700)
```sh
docker pull rabbitmq：management
docker run -dit --name Myrabbitmq -e RABBITMQ_DEFAULT_USER=admin -e RABBITMQ_DEFAULT_PASS=admin -p 15672:15672 -p 5672:5672 rabbitmq:managemen
```
[mq地址](http://167.179.88.93:15672)

### 刷新方式
* 利用消息总线触发一个客户端/bus/refresh,从而刷新所有客户端配置
* 利用消息总线触发服务端/bus/refresh,从而刷新所有客户端配置
> 选择方式2  因为破坏了其中某个微服务的单一性,对等性,局限性