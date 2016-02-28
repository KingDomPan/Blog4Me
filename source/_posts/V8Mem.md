---
title: V8的垃圾回收机制与内存限制
date: 2016-02-16 11:01:30
tags: [V8, Nodejs]
---

##### V8的对象分配
在V8引擎中, 64位系统下的限制内存大概是1.5G, 32位下大概是0.7G, 在这样的限制下, NOde无法直接操作大对象内存.
此限制来源于当初V8引擎的设计主要是为了浏览器使用, 在浏览器端, 改内存限制已满足使用. **深层次的限制是JS的单线程和V8的垃圾回收机制的限制**.
V8的垃圾回收, 一次小的垃圾回收需要50mm以上, 一次非增量式的垃圾回收至少需要1s以上, 会引起线程的暂停执行, 影响应用程序的性能.
所有的JS对象都是在堆上进行分配的, `process.memoryUsage();` 返回`已经申请到的堆内存`,`已经使用的堆内存`,`进程的常住内存`.
`node --max-old-space-size=1700 test.js` 单位MB **设置老生代的最大值**
`node --max-new-space-size=1024 test.js` 单位KB **设置新生代内存空间的大小**

<!-- more -->
##### V8的垃圾回收限制
- V8主要的垃圾回收算法
  - 分代式垃圾回收机制
  - 内存分代: 新生代, 老生代, **V8堆的大小几乎==新生代 + 老生代**
  - 新生代内存: 64位系统下的限制内存大概是16M, 32位下大概是8M. `reserved_semispace_size`
  - Scavenge算法: 分代的基础上, 新生代算法实现是Cheney(其实就是`内存复制法`)
  - 新生代对象晋升条件: 经过Scavenge算法回收, To空间的内存占用比超过限制
  - 老生代回收: Mark-Sweep 标记回收算法 Mark-Compact 标记压缩算法

##### 查看垃圾回收日志
`node --trace_gc -e "var a = []; for( var i = 0; i < 1000000; i++ ) { a.push(new Array(100)); }" >> gc.log` 从标准输出中打印垃圾回收日志

`node --prof` V8执行时的性能分析数据 获取`isolate-0x101804a00-v8.log`
`linux-tick-processor isolate-0x101804a00-v8.log`

##### 高效使用内存
- 作用域 **函数才会创建作用域**
- 作用域链
- 变量主动释放, `window.KM = undefined;`
- 闭包: 外部作用域访问内部作用域中变量的方法叫做闭包. **会造成原始作用域内存不会被释放**

##### 内存指标
- 内存使用情况
  - `process.memoryUsage();`
  - `os.totalmem();` `os.freemem();` 返回系统的总内存和闲置内存

##### 堆外内存
- `process.memoryUsage().rss > process.memoryUsage().heapTotal` 差值即为非V8分配的堆外内存
- Buffer分配在堆外内存上! 不受V8内存大小的限制

##### 内存泄露
- 缓存
- 队列消费不及时
- 作用域未释放

##### 内存泄露排查
- node-heapdump
- node-memwatch
