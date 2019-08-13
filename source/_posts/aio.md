---
title: 同步IO与AIO
date: 2018-12-10 11:22:45
tags: IO
---
### 背景
所有进程IO操作都分为2个阶段：

* 进程向内核发起IO请求，内核告诉data是否ready
* 内核操作IO外设，在内核空间和进程空间copy数据

BIO和NIO说的是第一阶段，内核的响应是阻塞的还是非阻塞的（是否阻塞进程），同步异步则说的是第二阶段（进程主动轮询IO是否就绪内核主动发送通知信号）。
### Unix 5 IO models
* blocking I/O
* nonblocking I/O
* I/O multiplexing
* signal-driven I/O
* asynchronous I/O	