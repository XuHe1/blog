---
title: VisualVM使用的坑
date: 2019-01-10 18:30:41
tags: 
		- tool
		- java
---

### 背景
面试高开的同学都知道，面试官一般会问jvm问题排查和优化：比如线上内存泄漏如何发现，jvm如何调优等等。2周前打算去面试一个java开发经理的岗位，友好的猎头把之前面试者的遇到的题目发给了我:
> 1、java内存模型；<br>
> 2、hashmap和arrayList扩容的原理；<br>
> 3、常用的线程池有哪些；<br>
> 4、数据库慢查询的优化；<br>
> 5、如何设计索引；<br>
> 6、数据库的事务隔离级别有哪些；<br>
> 7、mybatis 和hibernate的区别，使用场景；<br>
> 8、什么是内存泄漏，线程死锁；<br>
> 9、内存泄漏如何诊断。

### 常用工具
* jdk官方提供的
> 1. jconsole <br>
> 2. visualVM <br>

* 第三方的，如 MAT

### VisualVM使用遇到的坑
* 远程服务器启动jstatd，报错

	> Could not create remote object
	access denied ("java.util.PropertyPermission" "java.rmi.server.ignoreSubClasses" "write")
	java.security.AccessControlException: access denied ("java.util.PropertyPermission" "java.rmi.server.ignoreSubClasses" "write")
		at java.security.AccessControlContext.checkPermission(AccessControlContext.java:472)
		at java.security.AccessController.checkPermission(AccessController.java:884)
		at java.lang.SecurityManager.checkPermission(SecurityManager.java:549)
		at java.lang.System.setProperty(System.java:792)
		at sun.tools.jstatd.Jstatd.main(Jstatd.java:139)
		
* 查看oracle官网，或者 `man jstatd`

	```
	   grant codebase "file:${java.home}/../lib/tools.jar" {
	           permission java.security.AllPermission;
	       };
	
		...	
	   jstatd -J-Djava.security.policy=jstatd.all.policy
	```

* 启动visualVM，新建远程主机，无反应，可ping通，也可telnet，纠结了很久。

* jps 10.23.54.41

	```
	Error communicating with remote host: Connection refused to host: 127.0.0.1; nested exception is:
		java.net.ConnectException: Connection refused (Connection refused)
	```

* 启动jstatd，指定hostname后，成功连接到远程主机。

	```
	 jstatd -J-Djava.security.policy=tools.policy  -J-Djava.rmi.server.hostname=10.23.54.41  &
	```
* jps 成功， visualVM连接成功！

    ![](http://blog.lovezhangli.top/pics/jps.jpeg)

