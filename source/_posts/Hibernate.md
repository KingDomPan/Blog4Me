---
title: Hibernate N + 1, 二级缓存
date: 2015-07-24 19:21:12
tags: [缓存, N + 1]
---

#### Hibernate N + 1问题
- 问题描述
**Hibernate ManyToOne, 在做查询的时候, 由于N的一方默认的fetch=FetchType.EARGE, 所以会把关联对象一起查询出来, 造成一共查询N+1次**
- 解决方案
    - fetch=FetchType.LAZY, 这种方法在使用到对象的时候还是会发出select语句(延迟检索策略)
    - 使用session.createCriteria(Object.class)查询而不使用session.createQuery()
    - 使用BatchSize(size = 5), 只会发出(N + 1) / 5 条语句
    - 使用join fetch做关联查询(迫切左外连接策略)
    - 使用二级缓存加载所有对象, 在用Iterate来取对象的时候就只会发一个取id的sql来遍历循环二级缓存中的对象(只适用于session.list和session.iterate的N+1问题说明)

<!-- more -->

#### Hibernate 二级缓存
- hibernate.xml配置开启缓存策略
  - hibernate.cache.use_second_level_cache = true
  - hibernate.cache.region.factory_class = org.hibernate.cache.ehcache.EhCacheRegionFactory
  - hibernate.cache.provider_configuration_file_resource_path = ehcache.xml

- ehcache.xml配置缓存存储
```xml
<ehcache>
  <diskStore path="user.dir" />
  <defaultCache
        maxElementsInMemory="10000"
        eternal="false"
        timeToIdleSeconds="120"
        timeToLiveSeconds="120"
        overflowToDisk="true" />
  <cache name="com.xiaoluo.bean.Student"
        maxElementsInMemory="10000"
        eternal="false"
        timeToIdleSeconds="300"
        timeToLiveSeconds="600"
        overflowToDisk="true" />
  <cache name="sampleCache2"
        maxElementsInMemory="1000"
        eternal="true"
        timeToIdleSeconds="0"
        timeToLiveSeconds="0"
        overflowToDisk="false" />
</ehcache>
```
- xml和annotation的配置
  - cache = read-only
  - @Cache(usage=CacheConcurrencyStrategy.READ_ONLY)

- 开启hibernate的二级缓存的查询缓存配置
  - 只有当hql查询语句完全相同时, 连参数设置都要相同, 此时查询缓存才会有效果
  - 查询也能引起N + 1的问题, 因为查询缓存缓存的也仅仅是对象的id, 所以必须开启二级缓存配置
  - hibernate.cache.use_query_cache = true
  - setCacheable(true)
  - @Cacheable