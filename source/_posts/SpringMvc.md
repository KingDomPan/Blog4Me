---
title: SrpingMVC AOP 配置
date: 2014-01-12 10:21:12
tags: [Spring, MVC, AOP]
---

##### SpringMvc3.1 注解
- @Controller: 用于标识该类是处理器类
- @RequestMapping: 请求到处理器功能方法的映射规则
- @RequestParam: 求参数到处理器功能处理方法的方法参数上的绑定
- @ModelAttribute: 请求参数到命令对象的绑定
- @SessionAttributes: 用于声明session级别存储的属性, 放置在处理器类上
- @InitBinder: 自定义数据绑定注册支持, 用于将请求参数转换到命令对象属性的对应类型
- @CookieValue: cookie数据到处理器功能处理方法的方法参数上的绑定
- @RequestHeader: 请求头(header)数据到处理器功能处理方法的方法参数上的绑定
- @RequestBody: 请求的body体的绑定(通过HttpMessageConverter进行类型转换)
- @ResponseBody: 处理器功能处理方法的返回值作为响应体(通过HttpMessageConverter进行类型转换)
- @ResponseStatus: 定义处理器功能处理方法/异常处理器返回的状态码和原因
- @ExceptionHandler: 注解式声明异常处理器
- @PathVariable: 请求URI中的模板变量部分到处理器功能处理方法的方法参数上的绑定, 从而支持restful风格

<!-- more -->

##### 类型转换
- ConversionService 进行类型转换, PropertyEditor依然有效
- @NumberFormat 和 @DateTimeFormat 进行数字和日期的格式化
- HttpMessageConverter Http的输出输入转换器, 比如JSON/XML等的数据转换器
- ContentNegotiatingViewResolver 内容协商视图解析器, 还是视图解析器, 只是它根据请求信息响应到不同的视图
- SpringMvc的名称空间
	- `<mvc:annotation-driven>`
	- 自动注解, `DefaultAnnotationHandlerMapping` `AnnotationMethodHandlerAdapter`
	- 自动注解3.1 `RequestMappingHandlerMapping ` `RequestMappingHandlerAdapter`
	- ConversionService 自动注册
	- HttpMessageConverter 自动注册
	- `<mvc:interceptors>` 自定义拦截器
	- `<mvc:view-controller>` 收到请求直接选择相应的视图
	- `<mvc:resources>` 逻辑静态资源路径到物理静态资源路径的支持
	- `<mvc:default-servlet-handler>` 将静态资源文件转交给默认的servlet处理

#### Spring AOP
- 连接点(Jointpoint): 表示需要在程序中插入横切关注点的扩展点, 连接点可能是类初始化、方法执行、方法调用、字段调用或处理异常等等, Spring只支持方法执行连接点, 在AOP中表示为:在哪里干"

- 切入点(Pointcut): 选择一组相关连接点的模式, 即可以认为连接点的集合, Spring支持perl5正则表达式和AspectJ切入点模式, Spring默认使用AspectJ语法, 在AOP中表示为:在哪里干的集合"

- 通知(Advice): 在连接点上执行的行为, 通知提供了在AOP中需要在切入点所选择的连接点处进行扩展现有行为的手段: 包括前置通知(before advice)、后置通知(after advice)、环绕通知(around advice), 在Spring中通过代理模式实现AOP, 并通过拦截器模式以环绕连接点的拦截器链织入通知: 在AOP中表示为:干什么"

- 方面/切面(Aspect): 横切关注点的模块化, 比如上边提到的日志组件. 可以认为是通知、引入和切入点的组合: 在Spring中可以使用Schema和@AspectJ方式进行组织实现: 在AOP中表示为:在哪干和干什么集合"

- 引入(inter-type declaration): 也称为内部类型声明, 为已有的类添加额外新的字段或方法, Spring允许引入新的接口(必须对应一个实现)到所有被代理对象(目标对象), 在AOP中表示为:干什么(引入什么)"

- 目标对象(Target Object): 需要被织入横切关注点的对象, 即该对象是切入点选择的对象, 需要被通知的对象, 从而也可称为:被通知对象": 由于Spring AOP 通过代理模式实现, 从而这个对象永远是被代理对象, 在AOP中表示为:对谁干"

- AOP代理(AOP Proxy): AOP框架使用代理模式创建的对象, 从而实现在连接点处插入通知(即应用切面), 就是通过代理来对目标对象应用切面. 在Spring中, AOP代理可以用JDK动态代理或CGLIB代理实现, 而通过拦截器模型应用切面.

- 织入(Weaving): 织入是一个过程, 是将切面应用到目标对象从而创建出AOP代理对象的过程, 织入可以在编译期、类装载期、运行期进行.
