# 项目说明
## 简介

有朋友们来咨询关于用``Spring Cloud``全家桶开发微服务应用的问题、以及在``Kubernetes``上编排部署``Spring Cloud``微服务、如何寻址的问题，因此萌发了将我以前在几个互联网项目中开发和k8s部署微服务实战中积累的最佳实践提炼出来分享的想法。本项目为一个浓缩了Spring Cloud Microservices on Kubernetes最佳实践的mini微服务应用参考，用于最佳实践的分享，技术演示了：

1. 一个可快速复制的采用``Spring Cloud``全家桶来开发微服务应用的框架；
2. 一套可快速复制的将``Spring Cloud``微服务部署到``Kubernetes``集群上的部署方案和YAML部署脚本（在生产项目应用中，这些YAML脚本应该需要部署到公司的CI & CD pipeline上以实现自动化的CI & CD）。


## 技术栈

1. ``Kubernetes``和``Rancher Server``：作为自动化部署、伸缩和编排微服务容器集群的PaaS平台。

2. ``Spring Cloud``全家桶：用于开发微服务应用。采用目前最稳定的``Edgware RELEASE``，最佳搭配的全家桶组件有：
    - 开发：``Zuul``作为API Gateway，``Eureka``作为服务注册治理中心，``Spring Cloud Config``作为配置中心，``Hystrix``作为熔断器、降级和限流，``Ribbon``作为负载均衡，``Feign``作为声明式的REST Client。
    - 监控：``Spring Boot Actuator``+``Spring Boor Admin``提供每个微服务的自省和监控能力、以及可视化监控UI，``Hystrix Dashboard``用于可视化监控Hystrix Metrics，``Turbine``用于聚合各个微服务的Hystrix Metrics，``Sleuth``和``Zipkin``用于分布式调用跟踪。

3. ``Swagger`` & ``Swagger UI``：用于REST API文档和兼作REST API调试。

4. ``MyBatis``：用于``MySQL``数据库访问ORM。

5. ``RabbitMQ``：用于可靠的消息服务中间件。

6. ``Flyway``：秉承``DevOps``的``一切均代码``思想，采用``Flyway``用于数据库升级的配置管理。

7. 第三库：``lombok``，``guava``


## 在线演示

为方便朋友们“无痛”体验``Spring Cloud``微服务和``Kubernetes``集群部署，我在Azure云上购买了3台虚拟机，自己搭建了一套``Kubernetes``集群，并将该mini微服务演示项目和底层依赖部署到这套集群上。开放出一个read only的``Kubernetes``集群体验账号供大家体验。【注：因海外服务器网络访问延迟因素，已暂停Azure上的系统，正在将演示系统迁移到国内云服务器上，目前正在进行ICP备案号申请，备案成功后将继续开通该演示系统的域名访问】

``Kubernetes``集群体验账号：
- Web Console URL: master1.k8s.kyletiger.com
- Login Name: spring
- Password: spring123456

3台虚拟机的配置如下：
1. Kubernetes Master node：1台，配置为标准 F2s_v2 (2 vcpu，4 GB 内存)；
2. Kubernetes Worker node：2台，配置为标准 F1s (1 vcpu，2 GB 内存)。

请：不要流量攻击这套集群或漏洞扫描！这套系统只是纯粹用于交流和方便网友，小成本的部署环境而已，没有$去购买高性能虚拟机和WAF网关、抗DDoS等安全防护服务。


### 服务字典

注意：
1. Kubernetes集群里所有暴露出来可供外网访问的服务，旨在为了方便``Spring Cloud``爱好者在学习和调试时可远程使用现成的基础服务（例如，Eureka服务注册中心、配置中心），才特地采用``type: NodePort``部署方式额外再将该服务expose在外网IP上；
2. 对于实际项目，需采用``type: LoadBalancer``部署方案来expose服务，只对内网访问开放！

服务 | 内网服务寻址 | 内网服务端口 | 外网DNS寻址 | 外网服务端口
---|---|---|---|---
服务注册中心HA-0 | eureka-0.discovery.default.svc.cluster.local | 8761 | master1.k8s.kyletiger.com | 38761
服务注册中心HA-1 | eureka-1.discovery.default.svc.cluster.local | 8761 | node1.k8s.kyletiger.com | 38762
配置中心HA | 由服务消费者到Eureka Server上查询寻址方式 | 8888 | node1.k8s.kyletiger.com | 38888
API Gateway Zuul HA | 由服务消费者到服务注册中心上查询寻址方式 | 8080 | master1.k8s.kyletiger.com | 38080
Hystrix Dashboard | 由服务消费者到服务注册中心上查询寻址方式 | 9000 | node2.k8s.kyletiger.com | 39000
Hystrix Turbine | 由服务消费者到服务注册中心上查询寻址方式 | 9100 | master1.k8s.kyletiger.com | 39100
Spring Boot Admin | 由服务消费者到服务注册中心上查询寻址方式 | 9090 | node2.k8s.kyletiger.com | 39090
Employee微服务 | 由服务消费者到服务注册中心上查询寻址方式 | 8000 | node2.k8s.kyletiger.com | 38000
Department微服务 | 由服务消费者到服务注册中心上查询寻址方式 | 8100 | node2.k8s.kyletiger.com | 38100


- Kubernetes集群管理:

![](images/2018-12-13-02-50-08.png)

![](images/2018-12-16-21-28-39.png)

![](images/2018-12-16-21-29-59.png)

![](images/2018-12-13-03-07-37.png)

- 微服务注册中心：

![](images/2018-12-14-01-45-04.png)

- Spring Boot Admin:

![](images/2018-12-16-21-38-21.png)
![](images/2018-12-16-21-37-52.png)

- Hystrix Dashboard:

![](images/2018-12-14-01-49-17.png)

- Swagger:

![](images/2018-12-14-01-50-37.png)


## 更新计划

1. 采用``RabbitMQ``让各个微服务实例异步吐出Hystrix Metrics和``Turbine``进行异步采集聚合。
2. 部署上``Zipkin``和``Elasticsearch``：估计虚拟机的配置需要升级才能跑得动:(。
3. ``TCC``柔性分布式事务处理的演示：需要再开发几个演示``TCC``的微服务。
4. ``Spring Data JPA Repositories``+``Hibernate``：演示另一种数据库ORM方案。
5. 文档：增加&完善文档。


## 结语
感谢你的耐心阅读，如有对本项目中的Spring Cloud & Kubernetes的使用或者对本人的编码风格有更好的想法或者建议，欢迎通过邮件 <lisong.zheng@gmail.com>或QQ <40000646@qq.com>与我取得联系，万分感谢。
