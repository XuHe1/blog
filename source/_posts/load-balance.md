---
title: load-balance
date: 2018-10-10 13:21:41
tags: Architecture
---
### 概念
<pre>
* redundancy/failover: 冗余，故障转移
* HA： 高可用
* load-balance：负载均衡
* cluster： 集群
* master/slave: 主从
</pre>
>
The primary purpose of load balancing is to distribute the work load of an application onto multiple computers, so the application can process a higher work load. Load balancing is a way to scale an application.
>
A secondary goal of load balancing is often (but not always) to provide redundancy in your application. That is, if one server in a cluster of servers fail, the load balancer can temporarily remove that server from the cluster, and divide the load onto the functioning servers. Having multiple servers help each other in this way is typically called "redundancy". When an error happens and the tasks is moved from the failing server to a functioning server, this is typically called "failover".
>
A set of servers running the same application in cooperation is typically referred to as a "cluster" of servers. The purpose of a cluster is typically both of the above two mentioned goals: To distribute load onto different servers, and to provide redundancy / failover for each other.