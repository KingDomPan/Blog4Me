---
title: 工作原理 
date: 2016-03-14 11:23:35
tags: [DNS, HashMap]
---

- DNS域名解析
- HashMap内部原理

<!-- more -->

#### DNS域名解析
![DNS解析原理](/images/theory/dns.png)
1. 浏览器检查缓存中有没有域名对应的解析过的IP地址(浏览器缓存大小和时间都有限制TTL属性)
2. 查找操作系统中是否有这个域名对应DNS解析结果(hosts文件)
3. OS向网络配置中的DNS解析服务器进行查询(LDNS)
4. LDNS没命中, 直接到Root Server域名服务器请求解析
5. RServer返回给LDNS以个所查询的主域名服务器(gTLD Server -- 顶级域名服务器com cn org)
6. LDNS向gTLD服务器发送请求
7. 接受请求的gTLD服务器查找并返回此域名对应的Name Server域名服务器的地址, 这个Name Server就是域名的注册服务器
8. Name Server 域名服务器会查询存储的域名和IP的对应关系
9. 在正常情况下回返回目标IP记录连同一个TTL值给LDNS, LDNS进行IP和域名的关系缓存
10. LDNS把解析的域名返回给用户, 用户根据TTL值缓存在OS中

#### HashMap原理
![HashMap内存模型](/images/theory/map.png)
1. 初始化16大小的table数组, 每个数组对象都是Map.Entry类型
2. 初始化负载因子loadFactor大小为0.75
3. 每个Entry类型采用链表链接法来存放下一个对象
4. 对象的在数组中的索引位置根据key.hashCode的值再用hash函数计算出hash值, 在用indexFor计算出

![HashMap-put方法](/images/theory/put.png)
1. 对key做null检查, 如果为null就会被存储到table[0]中, 因为null的hash值总是为0
2. key的hashcode会被调用, 用indexFor计算出对象在数组中的存储索引位置
3. 对相同的key, 进行hash值和key值的双重校验(不同key会有相同的hash值), 如果满足条件用新值覆盖旧值并返回旧值
4. 新值入索引位置后, 采用next指针进行链接

![HashMap-get方法](/images/theory/get.png)
1. 对key做null检查, 如果是null, 返回table[0]位置上的元素
2. 根据key做hash值, 再计算出index索引位置
3. 取出索引位置上的元素进行迭代, **满足e.hash == hash && (e.key == key || e.key.equals(key))**则返回对应的值
