---
title: JAVA虚拟机垃圾回收(GC)
date: 2015-07-22 15:12:21
tags: [GC, JVM]
---

> 垃圾回收器常用的算法及实验原理

- 引用计数法
  - 为每一个对象分配一个整型的计数器
  - 无法处理循环引用的问题

- 标记-清除法(Mark-Sweep)
  - 标记阶段: 标记阶段首先通过根节点, 标记所有从根节点开始的较大对象
  - 清除阶段: 在清除阶段，清除所有未被标记的对象
  - 存在大量的内存碎片

<!-- more -->

- 复制算法
  - 现有的内存空间分为2块, 每次只使用其中一块, 垃圾回收时将正在使用的内存中的存活对象复制到另一块内存中, 之后, 清除正在使用中的内存中的所有对象, 最后, 交换2个内存的角色, 完成垃圾收集
  - 垃圾对象很多, 存活对象不是很多
  - 内存对半
  - JAVA新生代串行垃圾回收器, eden, from, to. from和to可以视为用于复制的2块大小相同的内存空间(survivor幸存者空间)

- 标记-压缩法(Mark-Compact)
  - 老年代更常见的情况是大部分对象都是存活对象
  - 这是一种老年代的回收算法, 在**标记清除**算法上进行了一些优化
  - 从根节点开始对可达到对象进行一次标记, 之后, **将存活对象压缩到内存的一端**, 再处理边界外的空间

- 增量算法
  - 垃圾回收时间过长, 会导致CPU消耗很高, APP挂起
  - 思想: 让GC线程和APP线程交替进行
  - 减少了系统的停顿时间, 但是带来了上下文的切换消耗, 会使得垃圾回收的成本增加, 造成系统吞吐量下降

- 分代
  - 将内存区间根据对象的特点分成几块, 使用不同的回收算法
  - Hot Spot
    - 将所有的新建对象都放入称为年轻代的内存区域, 年轻代的特点是对象会很快回收, 因此, 在年轻代就选择效率较高的复制算法
    - 新生代对象进入老生代, 老生代中的垃圾少, 回收代价低, 因此使用标记-压缩算法即可

##### 垃圾收集器类型
- 按线程数分: 串行垃圾回收和并行垃圾回收
- 按工作模式分: 并发式垃圾回收, 独占式垃圾回收(并发式与App交替进行)
- 按碎片处理方式可分为压缩式垃圾回收器和非压缩式垃圾回收器:
- 按工作的内存区间, 又可分为新生代垃圾回收器和老年代垃圾回收器:

##### 系统性能指标
- 吞吐量: 在应用程序的生命周期内, 应用程序所花费的时间和系统总运行时间的比值. 总运行时间=GC耗时 + app耗时
- 垃圾回收器负载: 和吞吐量相反, 垃圾回收器负载指来记回收器耗时与系统运行总时间的比值
- 停顿时间: 垃圾回收器运行时, app的暂停时间
- 垃圾回收频率: 垃圾收集器多久运行一次, 增大堆空间一般会减少频率次数, 但是可能会加大收集时间
- 反应时间: 当一个对象被回收后, 多长时间, 它占据的内存会被释放
- 堆分配: 合理的内存划分