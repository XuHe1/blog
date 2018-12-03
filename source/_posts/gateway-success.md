---
title: Spring cloud —— Zuul
date: 2018-10-01 11:28:11
tags:
    - MSA
    - java
    - spring cloud
---
在这普天同庆的日子，祝祖国生日快乐！

下面我宣布个重磅消息：

经过2天的努力，终于在昨天晚上8点多，成功搭建了公司的微服务架构，成就感满满，mark下，哈哈😄

下面就重点介绍下思路：
<!-- more -->
* 技术选型
	
	由于本人没有在阿里系的公司工作过，所以选用的是[spring-cloud-netflix](https://cloud.spring.io/spring-cloud-netflix/)这套，开玩笑的，有时间的话再试着学习Ali Dubbo，毕竟也是很牛逼的，进入了apache基金会。
	
	不吹了，切入正题，spring cloud这套还是博大精深的，涉及了micro-service很多东西，我只是用到了一部分，搭建了基本框架，有时间再挖掘完善。
	* eureka：服务注册发现用的就是它
	* zuul：网关路由
	* ribbon：负载均衡
* 实际路线
	
	* 用户相关服务
	
		由于mobile只传token（安全需要），因此我们的微服务的rest api无法使用，必须通过网关手动转发。
		如 wallet/{user_id}
	
	* 其他与用户无关的服务
	
		这些服务，就可以通过网关直接调用即可。
	
	* 非java 服务
	
		java服务可以自动注册到 Eureka Server，非java服务，如何服务注册呢？当然我们想到的是手动注册，而官方的确是支持[手动注册](https://github.com/Netflix/eureka/wiki/Eureka-REST-operations)

* TODO

	* eureka server多实例
	* spring cloud配置中心
	* ribbon
	* hystrix
		