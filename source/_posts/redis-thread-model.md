---
title: redis 线程模型
date: 2018-11-29 18:24:18
tags: Redis
---
* NIO两种方式

 > a thread loop ServerSocketChannel with a selector
 
 > a accept thread  + a worker thread with a selector

* 线程安全
 
 > 单线程，任何时刻，只能执行一个命令
 
 > 原子型，任何命令都是原子操作