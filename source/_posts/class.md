---
title: ClassLoader工作机制
date: 2016-03-14 21:42:40
tags: [JAVA, ClassLoader]
---

#### ClassLoader加载机制
- 将Class加载到JVM中
- 审核每个类应该由谁来加载

##### ClassLoader类结构分析
- `Class<?> defineClass(byte[]. int, int)` 将字节流解析成JVM能够识别的Class对象(意味着可以通过其他方式比如网络来生成Class对象)
- `Class<?> findClass(String)` 实现类的加载规则, 取得要加载类的字节码, 再调用defineClass
- `Class<?> loadClass(String)`
- `void resolveClass(Class<?>)`

<!-- more -->

##### ClassLoader的等级加载机制
- 上级委托接待机制
- Bootstrap ClassLoader: 加载JVM自身工作的类, 完全由JVM自己控制, 没有更高一级的父加载器, 也没有子加载器
- ExtClassLoader: JVM自身的一部分, 加载JVM的扩展类库 `System.getProperty("java.ext.dirs")`
- AppClassLoader: 父类是ExtClassLoader, 所有在`system.getproperty("java.class.path")`目录下的类都被这个加载器加载(classpath)
- 自定义类加载器(URLClassLoader): 父加载器都是AppClassLoader

##### 如何加载class文件
![How To Load Class](/images/class/classloader.png)
1. 找到.class文件并把这个文件包含的字节码加载到内存中
    1.1 `URLClassLoader -> URLClassPath -> URL(表示classpath)` 最终创建FileLoader或者JarLoader, 或者使用默认的加载器, 使用findClass()加载class到内存中
2. 字节码验证, class类数据结构分析和内存分配, 符号表链接
    2.1 字节码验证: 类装入器对于类的字节码要做许多检测, 以确保格式正确, 行为正确
    2.2 类准备: 在这个阶段准备代表每个类中定义的字段, 方法和实现接口所必须的数据结构
    2.3 解析: 在这个阶段类装入器装入类所引用的其他所有类, 可以用许多方式引用类, 如超类, 接口, 字段, 方法签名, 方法中的本地变量
3. 类中的静态属性和初始化赋值, 静态块的执行
    3.1 在类中包含的静态初始化器都被执行, 在这个阶段未被初始化的静态字段被初始化为默认值

##### 常见加载类错误分析
1. ClassNotFoundException: JVM要加载指定类的class文件时没找到指定的class文件, 检查当时的classpath. `this.getClass().getClassLoader().getResource("").toString()`
    1.1 `Class.forName()`
    1.2 `ClassLoader.loadClass()`
    1.3 `ClassLoader.findSystemClass()`
2. NoClassDefFoundException: 触发JVM隐式加载某些类时发现这些类不存在的异常(确保每个被引用的类都在classpath下)
    2.1 new 关键字
    2.2 属性引用某个类
    2.3 继承了某个接口或者类
    2.4 方法的某个参数中引用了某个类
3. ClassCashException: 程序中出现强直类型转换时出现这个错误(显式指定数据类型或者使用instanceof进行类型判断)
    3.1 对于普通类型, 对象必须是目标类型或者是目标类型的子类的实例
    3.2 对于数组类型, 目标类必须是数组类型或者`java.lang.Object, java.lang.Cloneable, java.lang.Serializable`

##### 常用的ClassLoader分析
![Tomcat-ClassLoader](/images/class/tomcat-classloader.png)
- Bootstrap类的`initClassLoaders`方法中通过ClassLoaderFactory的createClassLoader方法创建StandardClassLoader
- StandardClassLoader实际上只是代理了AppClassLoader进行Class的加载, 所以Tomcat的架构中的类加载器还是AppClassLoader
- StandardClassLoader要负责加载Tomcal容器中自己的ClassPath位置的类
- Web应用程序中的类是如何被Tomcat加载进来??
    1. 一个App在Tomcat中由一个StandardContext表示并来解析web.xml中的J2EE Web规范的对象
    2. StandardContext.startInternal()初始化时会检测loader属性是否存在. 不存在就使用getParentClassLoader()创建
    3. WebAppLoader -> WebAppClassLoader <- StandardWrapper.loadServlet() -> InstanceManager
    4. WebappClassLoader覆盖了父类的loadClass()方法使用了自己的类加载机制
