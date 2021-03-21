# 下一代SpringCloud技术栈

目前市场上主流的 第一套微服务架构解决方案：Spring Boot + Spring Cloud Netflix。 最近由于Netflix公司宣布Spring Cloud Netflix 系列技术栈进入维护模式，于是采用 Spring
Cloud Alibaba 方案来替代。

我们建议对这些模块提供的功能进行以下替换

CURRENT                      | REPLACEMENT
------------------------- | --------------------------------------
Hystrix                      | Sentinel/Resilience4j
Hystrix Dashboard/Turbine | Micrometer + Prometheus
Ribbon                    | Spring Cloud Loadbalancer
Zuul 1                    | Spring Cloud Gateway
Eureka                    | Nacos Discovery
Spring Cloud Config       | Nacos Config

出自[【官方新闻】Spring Cloud Greenwich.RC1 available now](https://spring.io/blog/2018/12/12/spring-cloud-greenwich-rc1-available-now)



