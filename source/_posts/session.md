---
title: 分布式session
date: 2016-03-25 17:24:40
tags: [session]
---

#### 分布式缓存
- `Replication Session`(`Session`复制): 将一台机器上的`Session`数据广播复制到集群中的其他机器上
- `Sticky Session`(粘性`Session`): 同一个会话中的请求必须被转发到同一个节点上
- 缓存集中式管理: 将`Session`存入分布式缓存集群中的某台机器上, 当用户访问不同节点时先从缓存中拿`Session`数据
- `Not Sticky Session`(非粘性会话): 每一次请求都可能转发到不同节点

<!-- more -->

#### Memcached-Session-Manager
- 支持`粘性会话`和`非粘性会话`
- 没有单点故障
- 能处理`tomcat`故障转移
- 能处理`memcached`故障转移
- 可插拔
- 允许异步存储`session`
- `session`被修改时, 才会发给`Memcache`

#### MSM的粘性模式原理
1. `Tomcat`为主`Session`, `Memcache`为副本`Session`
2. 请求到来时使用主`Session`, 若是有修改同步到副本`Session`
3. `Tomcat`挂了之后, 其他`Tomcat`从`Memcache`中查询`Session`

#### MSM的非粘性模式原理
1. `Tomcat`为中转`Session`, M1为`主Session`, M2为`备用Session`
2. 请求到来, 从`M2-->M1`的顺序加载Session
3. `Session`变化同步到`Memcache`, 清除`Tomcat`的`Session`

#### 基于Kryo的序列化方案

#### 基于`ZooKeeper`集群的分布式Session方案
- 解决基于Memcache方法的数据丢失问题, 引入ZK的持久化存储介质
- ZK的一致性复制(在多个副本间保证数据的强一致性)和容错能力
![ZK-Session-持久化](/images/session/session.png)
