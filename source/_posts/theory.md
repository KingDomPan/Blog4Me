---
title: 工作原理 
date: 2016-03-14 11:23:35
tags: [DNS]
---

##### DNS域名解析
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
