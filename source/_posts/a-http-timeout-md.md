---
title: 记一次httpClient超时引发的消息堆积
date: 2019-08-02 13:48:45
tags: 
	- httpClient
    - rabbitmq
---


消费线程中调用了第三方的接口, 由于第三方接口宕机, 导致连接超时.

```
    connnect timeout
```

<!-- more -->
