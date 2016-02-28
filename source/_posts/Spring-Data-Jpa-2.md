title: Spring-Data-Jpa-MVC
date: 2014-12-11 10:42:56
categories: Spring-Data
tags: [Spring, Data, Jpa]
---

这节讲解一下SpringMVC和SpringDataJpa的一点关系, 主要是DomainClass的自动注入和方法**分页排序**参数的自动解析.

<!-- more -->

##### DomainClassConverter
配置这个Converter可以通过请求参数或者路径变量, 解析出实体对应的repository在数据库中的实体对象, 只要repository中有对应的查询方法.
```java
@Controller
@RequestMapping("/users")
public class UserController {
  @RequestMapping("/{id}")
  public String showUserForm(@PathVariable("id") User user, Model model) {
    model.addAttribute("user", user);
    return "userForm";
  }
}
```
###### DomainClassConverter配置
```xml
<!-- 类型转换及数据格式化 -->
<bean id="conversionService" class="org.springframework.format.support.FormattingConversionServiceFactoryBean"/>

<!-- 直接把id转换为entity 必须非lazy否则无法注册 -->
<bean id="domainClassConverter" class="org.springframework.data.repository.support.DomainClassConverter">
    <constructor-arg ref="conversionService"/>
</bean>

<!-- MVC注册 -->
<mvc:annotation-driven conversion-service="conversionService" />
```

##### HandlerMethodArgumentResolver接口
HandlerMethodArgumentResolver: 从请求对象中解析出Pageable and Sort实例.
- PageableHandlerMethodArgumentResolver
    - **page**
    - **size**
- SortHandlerMethodArgumentResolver
    - **sort** like sort=firstname&sort=lastname,asc

```java
@Controller
@RequestMapping("/users")
public class UserController {
  @Autowired UserRepository repository;
  @RequestMapping
  public String showUsers(Model model, Pageable pageable) {
    model.addAttribute("users", repository.findAll(pageable));
    return "users";
  }
}
```

###### HandlerMethodArgumentResolver配置(官方的, 无效啊)
```xml
<!-- 方法处理的映射适配器 -->
<bean class="org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter">
    <property name="messageConverters">
        <list>
            <ref bean="byteArrayConverter"/>
            <ref bean="stringConverter"/>
            <ref bean="resourceConverter"/>
            <ref bean="sourceConverter"/>
            <ref bean="xmlAwareFormConverter"/>
            <ref bean="jaxb2RootElementConverter"/>
            <ref bean="jacksonConverter"/>
        </list>
    </property>
    <!-- 定义Spring-data的分页和排序实体 -->
    <property name="customArgumentResolvers">
        <list>
            <bean class="org.springframework.data.web.PageableHandlerMethodArgumentResolver" />
        </list>
    </property>
</bean>
```

###### HandlerMethodArgumentResolver配置(有效)
```xml
<!-- 注解驱动 -->
<mvc:annotation-driven>
    <mvc:argument-resolvers>
        <bean    class="org.springframework.data.web.PageableHandlerMethodArgumentResolver" />
    </mvc:argument-resolvers>
</mvc:annotation-driven>
```
