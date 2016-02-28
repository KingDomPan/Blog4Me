---
title: Redis
date: 2015-07-24 20:44:39
tags: [缓存, 数据库, Redis]
---

##### 什么是Redis
- KV Store 用来补充关系型数据库
- 可以持久化的cache, 可以保存一些频繁访问的数据
- 高性能的内存数据库
- 数据结构服务器, 可以保存复杂的数据类型
- 缺点: 数据库容量受到物理内存的限制, 不能用作海量数据的高性能读写, 并且没有原生的可扩展机制, 要依赖客户端实现分布式读写, 因此适合场景主要局限在较小量的高性能操作和运算上

<!-- more -->

##### Redis为什么比Memcached快
- redis带有持久化功能, 后者是完全基于内存的
- CAS问题, Memcached中防止竞争修改资源的一种手段, 需要为每个key设置一个隐藏的cas token, 相当于版本号, 在数据量上大幅度提升的时候会有性能上的差别
- 单台Redis的存放数据必须比物理内存小, 冷热数据的分离, 基于持久化技术, Redis2.0的VM概念
- Redis的VM实现是重复造轮子, VM的实现不是基于OS原有的page的概念, 缩小4k的粒度控制
- 用get/set方式使用Redis, 复用key, 使得原有的相同的内存空间可以存放100倍以上的数据
- 使用aof代替snapshot(存储方式), 默认的是snapshot, 实现方法是定时将内存的快照数据持久化到硬盘, 缺点是crash之后会出现数据丢失的情况. aof模式(append only mode), 在写入内存数据的同时将操作命令保存到日志文件(binlog), (其实这样无法保证数据的原子操作性), 使用Replicatio来代替, 复制基本没有延迟
- Redis作者说, 平均到单个核上, 在单条数据不大的情况下, Redis的性能会更好. Redis是单线程的, 只能使用一个CPU内核, 而Mem是多线程的, 所以对于一个实例来说, 性能上肯定是Mem占据优势.
- 所以性能上不是2个最主要的差别!!!

##### Redis数据类型
- string 字符串 (set get incr decr mget)
	- 获取字符串长度
	- 往字符串里append
	- 设置和获取某一段的内容
	- 设置和获取字符串的某一位
	- 批量设置一系列字符串的内容
	- 使用场景: 不仅仅是stirng, 也可以是数字, 比如ip, 通过原子递增保持计数
- hash 散列表, 其实就是字典 (hset hget hgetall)
	- 应用场景1: 用户信息 id: {name: '', age: '', birthday: ''}
- list 列表 lpush,rpush,lpop,rpop,lrange,BLPOP(阻塞版, 主要是为了避免轮询)
	- 应用场景1: 最新消息的排行版
	- 应用场景2: 实现消息队列
	- 实现机制: 双向链表(发送缓冲队列)
- set 无序集合 (sadd, srem, spop, sdiff, smembers, sunion)
	- 自动排重功能
	- 提供了判断一个成员是否在一个set集合里面的接口
	- 应用场景1: 比如微博中, 每个人的好友列表都存放在一个set中, 这样求2个人的共同好友, 只要求一下交集
	- 交集, 并集, 差集
	- 实现方式: 内部是一个value永远为null的HashMap, 实际上就是通过计算hash的方式来快速排重的, 这也是set提供判断一个成员是否在集合中的原因.
- zset 有序集合 (zadd, zrange, zrem, zcard)
	- 应用场景1: 以某个条件为权重, 比如按顶的次数排序
	- ZREVRANGE命令可以用来按照得分来获取前100名的用户, ZRANK可以用来获取用户排名, 非常直接而且操作容易
	- 定义优先级自动排序
	- 需要精准设置过期时间的应用: 按照过去时间来清除redis中的数据, 设置可以把该字段当成是数据中的索引来删除数据库中的字段.
	- 实现原理: 内部使用HashMap和跳跃表(SkipList)来保证数据的存储有有序, HashMap里面放的是成员到Score(优先级)的映射, 而跳跃表里面存放的是所有的成员, 排序依据是HashMap里存的score, 使用跳跃表的结构可以获得比较高的查找效率, 并且在实现上比较简单

##### Redis内存存储原理
- string
	- string存储原理
		- 二进制安全的, 可以存放一个图片或者对象的序列化值 maxSize = 512M
		- int len; int free; char buf[]; 长度, 剩余长度, 实际存储位置
			- 字符串拼接处理策略
				- 初始化数据, free = 0
				- append长度为K的数据, 重新分配空间, free大小为K+N+1, N为len
				- 再次append, 如果K1+1&lt;free, 不再重新分配, 否则执行上一条策略
	- string使用场景
		- 使用INCR命令作为原子计数器
		- APPEND追加字符串
		- 作为GETRANGE和SETRANGE的访问向量
		- 小空间编码大数据, GETBIT 和 SETBIT创建一个Redis支持的Bloom过滤器
- hash
	- hash存储原理(每个字典使用2个hash, 用于实现渐进式的rehash)
		- 实际上就是Java中的HashMap的实际, 只不过是基于C语句的实现
		- rehash实际上是一次表的扩容, 或者说是修改表的容量, 将ht(0)复制到ht(1), 并将1作为0的过程
		- ![rehash](/images/Redis/rehash.png)
- list
	- list存储原理
		- 一个列表做多包含(2^32)-1个元素, 访问列表头尾元素超级快, 中间部分需要O(n)
		- listNode *head, *tail; long len;  复制函数, 释放函数, 比对函数
		- **使用双端链表**
			- 事务模块中来按顺序保存输入的命令
			- 服务器模块中来保存多个客户端
			- 订阅/发送模块来保存订阅模式的多个客户端
			- 事件模块来保存事件时间(time event)
	- 使用场景
		- 社交网络中建立一个线性的时间模型, LPUSH用户的最新时间元素, 使用LRANGE去接收一些最近插入的元素
		- 使用LPUSH和LTRIM去创建一个永远不会超过指定元素数据的列表, 但是记住最后的N个元素
		- 列表被用来作为消息传递(消息队列)
		- BLPOP命令, 避免轮询
- set
	- set存储原理
		- O(1)完成元素的删除和添加, O(1)测试是否包含该元素, 最多支持(2^32)-1个元素
	- 使用场景
		- 追踪一件事情, 比如记录访问IP
		- 擅长表现关系
		- 使用SPOP或者SRANDMEMBER命令随意抽取元素
- zset
	- zset存储原理
		- hashMap + 跳跃表
		- 以非常快的速度(O(log(N)))添加, 删除, 更新元素
		- 元素是有序的, 所以可以根据score很快的获取一个元素的范围
		- 可以完成很多对性能有极端要求的任务
	- zset使用场景
		- 在线游戏中可以ZRANK和ZRANGE查看顶级用户或者用户的排行榜
		- 有序集合常常被用来索引存储在Redis中的数据

##### Redis其他功能
- 消息订阅: 对一个key进行了消息发布后, 所有订阅它的客户端都会收到相应的消息, 最主要的应用场景就是普通的即时聊天, 群聊等.
- subscribe rain
- publish rain 'my love is ynn' 返回订阅该key的客户端个数

##### Redis事务
- **transaction**
- **watch** 对key进行watch, 如果发现了更改, 那么transaction就会发现并拒绝执行
![redis的检测, 事务, 执行](/images/Redis/redisWatch.png)

##### Redis事务原理
- 事务用法: multi exec命令包围, 处在这2条命令中的一条或者多条命令, 会以FIFO的方法运行
- Redis的事务并不保证关系型数据库的ACID性质, 因此因为服务器失败而造成的数据不一致会存在
- 事务原理:
	- MULTI
		- multiCommand 对redisClient结构的flags进行检查和设置
			- 首先检查flags, 确保没有嵌套使用MULTI命令
			- 检查通过, 那么就用位操作, 将REDIS_MULTI这个FLAG打开
			- 最后向客户端返回ok
		- processCommand(redisClient *c) 处理命令, 如果命令不是EXEC, discard, multi, watch命令, 那么命令入队列queueMultiCommand(redisClient *c)
		- queueMultiCommand(redisClient *c)函数将要执行的命令, 命令的参数个数以及命令的参数放进multiCmd结构中, 并将这个结构保存到redisClient.mastate.command数组的末尾, 从而形成了一个保存了要执行的命令的FIFO队列信息![事务命令数组](/images/Redis/trans.png)
		- execCommand(redisClient *c)执行事务
			- 如果没执行过MULTI就报错
			- 如果在执行事务前, 有监视key的改变(watch), 那么取消事务
			- 为了保证事务的一致性和原子性
				- 如果处在AOF模式中, 向AOF文件发送MULTI
				- 如果处在复制模式中, 向附属结点发送MULTI
		- 开始执行事务中的所有命令
		- 恢复所有的参数和命令
		- 释放事务资源和重置事务状态 freeClientMultiState initClientMultiState
	- 取消事务(discard)命令实现
		- discardTransaction 释放事务资源和重置事务状态, 关闭FLAG, 取消所有对KEY的监控
		- discardCommand 放弃执行事务命令, 如果没有执行MULTI, 那么就报错, 否则调用 discardTransaction, 最后返回OK