---
title: What is JavaEE
date: 2019-08-02 16:00:25
tags:
	- J2EE
---

做过java开发的都听说过JavaEE,  招聘信息里也经常会看到: 熟悉J2EE规范,  那什么是JavaEE?

<!-- more -->

Oracle文档中[javaee5](https://docs.oracle.com/javaee/5/firstcup/doc/gcrky.html#gcrkr)文档中给出了权威的说法:

> java技术既是一门编程语言,又是一个平台, java编程语言是一个高级别的面相对象的语言,有着它自己独有的语法和风格, Java平台是基于java语言的应用运行的环境.

JavaEE是Java编程语言**平台**之一(另外两个是, JavaSE和JavaME),  所有的平台包括:

* JVM: 运行java应用的**程序**
* API: 组件(**component**)的集合, 组件可以用来创建其他组件或应用.

 JavaEE平台提供API和运行环境,用来构建可扩展、多层级、可靠、安全的网络应用.

多层级: 

* Client 层:  浏览器、标准应用、一个其他的服务

* Web 层:  Servlet、 Contexts and DI、 JSP

* Business 层:  EJB、JAX-RS restful、JPA entities

* EIS 层:  enterprise information systems ,  包括: 数据库服务器、资源计划等,  JDBC,  JPA,  Connector Architecture, JTA