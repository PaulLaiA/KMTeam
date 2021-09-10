---
界: Spring Framework
门: Spring Cloud
纲: 
tags: ["#SpringFramework/#SpringCloud","#MicroServices","#BQ"]
aliases:
  - 
date: 2021-09-08
---

Eureka：注册中心，Eureka Client会将服务注册到Eureka Server，并且Eureka Client还可以反过来从Eureka Server拉取注册表。

Ribbon：服务发起请求时，基于Ribbon做负载均衡，从一个服务的多台机器中选择一台。

Feign：基于Feign的动态代理机制，根据注解和选择的集群，拼接请求URL地址，发起请求。

Hystrix：发起请求通过Hystrix线程池来走的，不同服务走不同的线程池，实现不同服务调用的隔离，避免服务雪崩问题。

Gateway：前端、移动端调用后端系统，统一从Gateway网关进入，有网关转发请求到对应的服务。