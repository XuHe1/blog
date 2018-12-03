---
title: Eureka-HA
date: 2018-10-02 10:55:56
tags: MSA
---
好吧，我承认，我一旦学习起来会上瘾的，连我自己都害怕（此处有装逼嫌疑）！

昨天搭建的微服务框架，最后留了个TODO LIST里面提到HA(high Availability)，最后没控制住自己，晚上还是搞起来了。

刚刚跑起来，趁着热，就写进了博客，记录下再这个过程踩过的坑：
<!-- more -->

[官方的文档](http://cloud.spring.io/spring-cloud-netflix/1.4.x/single/spring-cloud-netflix.html#spring-cloud-eureka-server-zones-and-regions)指出配置HA关键：

<p>
	By default every Eureka server is also a Eureka client and requires (at least one) service URL to locate a peer. If you don’t provide it the service will run and work, but it will shower your logs with a lot of noise about not being able to register with the peer.

</p>
是时候展现英语功底的时候了（开玩笑的，英语不怎么太好的童鞋，可以粘贴到google翻译），主要意思就是：Eureka Server同时也是Eureka Client，配置对等节点时，只需要像我们注册服务时，配置个serviceUrl指向对等节点地址即可。

下面开始动手，我是在1台机器起了2个实例，我的配置文件:

```
---
spring:
  application:
    name: discovery
    version: 1.0.0
    env: test

server:
  port: ${PORT:8761}

eureka:
  environment: ${ENV:test}
  datacenter: ${DATACENTER:master}
  instance:
    hostname: 127.0.0.1
    leaseRenewalIntervalInSeconds: ${LEASE-TTL:30}
  client:
    registerWithEureka: true
    fetchRegistry: true
    serviceUrl:
      defaultZone: http://127.0.0.1:8762/eureka/
  server:
    waitTimeInMsWhenSyncEmpty: 0
    enable-self-preservation: false # 关闭安全模式
    eviction-interval-timer-in-ms: 60000 # 清理间隔
    
---
spring:
  application:
    name: discovery
    version: 1.0.0
    env: test

server:
  port: ${PORT:8762}

eureka:
  environment: ${ENV:test}
  datacenter: ${DATACENTER:master}
  instance:
    hostname: 127.0.0.1
    leaseRenewalIntervalInSeconds: ${LEASE-TTL:30}
  client:
    registerWithEureka: true
    fetchRegistry: true
    serviceUrl:
      defaultZone: http://127.0.0.1:8761/eureka/
  server:
    waitTimeInMsWhenSyncEmpty: 0
    enable-self-preservation: false # 关闭安全模式
    eviction-interval-timer-in-ms: 60000 # 清理间隔

```

成功启动后，只有8762端口上能看到DS Replicas，8761端口没有DS Replicas，我怀疑HA没有搭建成功，果然：
![](http://blog.lovezhangli.top/pics/cancel-service.jpg)
8762上服务正在evict，显然两个实例没有正常通信，两个实例使用的hostname都是127.0.0.1，修改主机名：

```
	127.0.0.1   node1 node2
```
重新修改配置：

```
spring:
  application:
    name: discovery
    version: 1.0.0
    env: test

server:
  port: ${PORT:8761}

eureka:
  environment: ${ENV:test}
  datacenter: ${DATACENTER:master}
  instance:
    hostname: node1
    leaseRenewalIntervalInSeconds: ${LEASE-TTL:30}
  client:
    registerWithEureka: true
    fetchRegistry: true
    serviceUrl:
      defaultZone: http://node2:8762/eureka/
  server:
    waitTimeInMsWhenSyncEmpty: 0
    enable-self-preservation: false # 关闭安全模式
    eviction-interval-timer-in-ms: 60000 # 清理间隔
    
---
spring:
  application:
    name: discovery
    version: 1.0.0
    env: test

server:
  port: ${PORT:8762}

eureka:
  environment: ${ENV:test}
  datacenter: ${DATACENTER:master}
  instance:
    hostname: node2
    leaseRenewalIntervalInSeconds: ${LEASE-TTL:30}
  client:
    registerWithEureka: true
    fetchRegistry: true
    serviceUrl:
      defaultZone: http://node1:8761/eureka/
  server:
    waitTimeInMsWhenSyncEmpty: 0
    enable-self-preservation: false # 关闭安全模式
    eviction-interval-timer-in-ms: 60000 # 清理间隔
```

重启之后成功，我就不贴图了（博客还没加CDN）

好了，今天下午真的不搞了，中午吃个饭，下午去滴水湖～～