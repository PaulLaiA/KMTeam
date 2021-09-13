---
界: Spring Framework
門: Spring Cloud
綱: 
tags: ["Spring","#SpringCloud","#MicroServices","#BQ"]
aliases:
  - 
date: 2021-09-08
---

Eureka：註冊中心，Eureka Client會將服務註冊到Eureka Server，並且Eureka Client還可以反過來從Eureka Server拉取註冊表。

Ribbon：服務發起請求時，基於Ribbon做負載均衡，從一個服務的多臺機器中選擇一臺。

Feign：基於Feign的動態代理機制，根據註解和選擇的集群，拼接請求URL地址，發起請求。

Hystrix：發起請求通過Hystrix執行緒池來走的，不同服務走不同的執行緒池，實現不同服務呼叫的隔離，避免服務雪崩問題。

Gateway：前端、移動端呼叫後端系統，統一從Gateway閘道器進入，有閘道器轉發請求到對應的服務。