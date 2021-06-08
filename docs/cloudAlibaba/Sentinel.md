# Sentinel
> 无需手动搭建监控平台，没有web界面进行配置，实现服务降级与限流

## 部署
### docker
```sh
docker pull bladex/sentinel-dashboard
docker run --name sentinel -d  -p 8858:8858  bladex/sentinel-dashboard
```
> 访问地址 http://10.4.7.131:8858/

### window
> 下载地址 https://github.com/alibaba/Sentinel/releases

`
java -jar xxx.jar
`
> 访问地址 http://localhost:8080/


### 项目
> cloudalibaba-sentinel-service8401


## 流控规则
![规则1](../static/alibaba/流控详细说明1.png)
![规则2](../static/alibaba/流控详细说明2.png)
### 线程数
> 在程序内控制并发，只提供一个线程
