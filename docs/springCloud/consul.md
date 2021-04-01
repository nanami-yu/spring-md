> [中文文档地址](http://springcloud.cc/spring-cloud-consul.html)
> * cloud-providerconsul-payment8006
> * cloud-consumer-service80
## 安装
```sh
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo

sudo yum -y install consul

#后台运行命令
nohup consul agent -server -ui -bootstrap-expect=1 -client=0.0.0.0 -bind 167.179.88.93 -data-dir=/consul/data >> /consul/logs/consul.log &
```
>pom依赖
```sh
        <!--consul-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-consul-discovery</artifactId>
        </dependency>
```
>yml
```yml
server:
  port: 8006

spring:
  application:
    name: cloud-provider-payment
  cloud:
    consul:
      host: 167.179.88.93
      port: 8500
      discovery:
        service-name: ${spring.application.name}
        heartbeat:
          enabled: true

```