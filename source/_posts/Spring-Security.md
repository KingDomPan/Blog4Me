title: Spring Security 安全
date: 2014-12-06 21:16:30
categories: Spring
tags: [Spring Security 安全验证]
---

项目需要, 开始了一把手的把玩, 首先拿SpringSecurity开刀

### 1. Java安全验证框架
- Apache Shiro
- Spring Security

### 2. 安全验证的2个主要概念
- 认证: 验证用户是否合法
- 授权: 合法的用户能够访问什么资源

<!-- more -->

### 3. Spring Security 基本配置

#### 3.1 Web.xml 配置SpringSecurity过滤器
![](/images/Spring-Security/web.jpg)

#### 3.2 配置SpringSecurity容器上下文环境, application-security.xml

##### 3.2.1 硬编码在xml文件中进行认证与授权(不推荐),**注意规定的数据格式**
![](/images/Spring-Security/hard.jpg)
这是Spring-Security常见Demo的配置方式.配置方式一目了然.
1. 定义admin用户,拥有ROLE_USER. ROLE_ADMIN权限
2. 定义user用户, 拥有ROLE_USER权限
3.  /admin.jsp 只能拥有ROLE_ADMIN的用户进行访问
4.  /\*\* 拥有ROLE_USER的用户均可访问
注: /\*\*是ant风格的通配符路径风格 **代码任意多层次路径,
如/admin/index.jsp  /admin/get/student/1 等等

```xml
<authentication-manager>标签:
    org.springframework.security.authentication.AuthenticationManager 接口
<authentication-provider>标签:
    org.springframework.security.authentication.AuthenticationProvider 接口
<user-service>标签:
    org.springframework.security.core.userdetails.UserDetailsService接口.进行用户认证的关键.
```

**<intercept-url>标签和<user>标签其实就是合法认证的用户和可授权的资源及其两者之间的关系的定义.
在使用其他配置方式的话, 关键就是如何获取这两部分数据, 并组装成框架能认识的格式.**

##### 3.2.2 jdbc的配置方式(采用数据库表)
###### 3.2.2.1 默认配置
![](/images/Spring-Security/default.jpg)
**对应的表结构**
![](/images/Spring-Security/default-table.jpg)
**Spring Security初始化会从这两张表中获得用户信息和对应权限.**

###### 3.2.2.2 自定义表配置,并提供认证方式所需要的sql和访问资源所需要的权限信息
**表结构如下**
![](/images/Spring-Security/table.png)
**JDBC认证并提供sql**
![](/images/Spring-Security/jdbc.png)

###### 3.2.2.3 重写SpringSecurity接口, 自定义实现认证与授权
**综合上面来看, 要实现认证和授权操作,
实际上就是解决用户, 角色, 三者的关系即可**
![](/images/Spring-Security/table-strcut.png)
t_acl_user 用户表
t_acl_role 角色表
t_acl_user_role 用户角色表
t_acl_permission 菜单/资源表
t_acl_role_permission 资源/角色表

上述三个表形成的多对对关系即可实现SpringSecurity的权限认真功能.
当然,也可以配置以下表关联, 但是从SpringSecurity需要数据的角度来说,上述三个表足够
t_acl_user_usergroup 用户-用户组关联表
t_acl_usergroup 用户组
t_acl_usergroup_role 用户组/角色关联表

---

#### 3.3 SpringSecurity提供的关键接口.
```java
org.springframework.security.core.GrantedAuthority
```
该接口只有一个方法String getAuthority();标识用户所拥有的权限
```java
org.springframework.security.access.ConfigAttribute
```
该接口只有一个方法String getAttribute();标识访问某资源需要的权限.
```java
org.springframework.security.access.AccessDecisionManager 访问决策管理器
关键方法
void decide(Authentication authentication, Object object,
Collection<ConfigAttribute> configAttributes)
throws AccessDeniedException, InsufficientAuthenticationException;
用来判断某个用户是否有权限访问某个资源
```
```java
org.springframework.security.core.Authentication接口,标识一个认证对象.
接口方法
Collection<? extends GrantedAuthority> getAuthorities();获取对象权限集合
Object getPrincipal(); 获取认证对象信息
```
```java
org.springframework.security.core.userdetails.UserDetailsService
```
**UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;
这个接口很关键,用户认证就是实现了这个接口**
```java
org.springframework.security.core.userdetails.UserDetails接口, 提供用户详细信息.
```
```java
org.springframework.security.access.vote.AffirmativeBased
org.springframework.security.access.vote.ConsensusBased
org.springframework.security.access.vote.UnanimousBased
```
上述三个类是`AccessDecisionManager`的间接子类, 提供三种不同的决策来决定某个用户是否能访问某个资源.
三个类分别代表
`AffirmativeBased` 只要有一个权限符合即可访问
`ConsensusBased` 只要有一个权限不符合就不能访问
`UnanimousBased` 弃权处理
在三者的内部是用过`AccessDecisionVoter`这个接口来实现的
    int vote(Authentication authentication, S object, Collection<ConfigAttribute> attributes);
可实现这个接口, 制定不同的投票器来决定用户是否有权限访问资源
也可直接实现AccessDecisionManager.
```java
org.springframework.security.web.access.intercept.FilterInvocationSecurityMetadataSource
```
接口方法`Collection<ConfigAttribute> getAttributes(Object object) throws IllegalArgumentException;`
该方法规定了如何获取被保护的资源及其权限的集合定义, object代表被保护的资源
```java
org.springframework.security.access.intercept.AbstractSecurityInterceptor
```
授权拦截器 `InterceptorStatusToken beforeInvocation(Object object);`
用户认证与资源授权的操作全部在这里实现.

#### 3.4 OOPS中的SpringSecurity重写
**代码所在包: com.ffcs.itm.oops.security.impl**
```java
OopsAccessDecisionManager 实现 AccessDecisionManager
OopsAuthorizationFilter 实现 AbstractSecurityInterceptor
OopsUserDetailServiceImpl 实现 UserDetailServiceImpl
OopsSecurityMetadataSource 实现 FilterInvocationSecurityMetadataSource
```
**另外有一些验证失败, 验证成功等的操作,如**
```java
OopsAuthenticationFailureHandler 实现 AuthenticationFailureHandler 认证失败处理
OopsAuthenticationSuccessHandler 实现 AuthenticationSuccessHandler 认证成功处理
OopsLogoutHandler 实现 LogoutSuccessHandler 注销后处理
```

#### 3.5 SpringSecurity标签库(可进行菜单的权限控制)
```jsp
<!--标签库-->
<%@ taglib prefix="sec" uri="http://www.springframework.org/security/tags"%>

//获取当前用户名
<sec:authentication property="name"/>

//获得当前用户所有的权限, 把权限列表放到authorities
<sec:authentication property="authorities" var="authorities" scope="page"/>
<c:forEach items="${authorities}" var="authority">
    ${authority.authority}
</c:forEach>

//authorize用来判断当前用户的权限，然后根据指定的条件判断是否显示内部的内容

//所有权限都需要满足才能进行操作
<sec:authorize ifAllGranted="ROLE_ADMIN,ROLE_USER">
  admin and user
</sec:authorize>

//只要有一个验证通过即可
<sec:authorize ifAnyGranted="ROLE_ADMIN,ROLE_USER">
  admin or user
</sec:authorize>

//当前没有, 则通过
<sec:authorize ifNotGranted="ROLE_ADMIN">
  not admin
</sec:authorize>

//acl/accesscontrollist 用于判断当前用户是否拥有指定的acl权限
<sec:accesscontrollist domainObject="${item}" hasPermission="8,16">
    <a href="message.do?action=remove&id=${item.id}">Remove</a>
</sec:accesscontrollist>
```
#### 3.6 SpringSecurity的方法级别权限
##### 3.6.1 控制全局范围的方法权限 global-method-security + protect-point
**maven添加对应的依赖**
```xml
    <dependency>
      <groupId>cglib</groupId>
      <artifactId>cglib-nodep</artifactId>
      <version>2.2.2</version>
    </dependency>
    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjrt</artifactId>
      <version>1.7.1</version>
    </dependency>
    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjweaver</artifactId>
      <version>1.7.1</version>
</dependency>
```
**Spring-Security的配置文件**
```xml
<global-method-security>
        <protect-pointcut
            expression="execution(* MesageServiceImpl.admin*(..))"
            access="ROLE_ADMIN"/>
</global-method-security>
```

##### 3.6.2 控制bean内的某个方法 bean中嵌入intercept-methods和protect标签
```xml
<beans:bean id="messageService" class="MessageServiceImpl">
    <intercept-methods>
        <protect access="ROLE_ADMIN" method="userMessage"/>
    </intercept-methods>
</beans:bean>
```

##### 3.6.3 使用annotation控制方法权限
**使用@Secured或者jsr250规范中定义的注解**

###### 3.6.3.1 Annotation
**启用Annotation配置**
```xml
<global-method-security secured-annotations="enabled"/>
```
**方法签名上使用**
```java
@Secured({"ROLE_ADMIN", "ROLE_USER"})
```

###### 3.6.3.2 jsr250
**Spring-Security配置**
```xml
<global-method-security secured-annotations="enabled" jsr250-annotations="enabled"/>
```
**maven依赖**
```xml
<dependency>
    <groupId>javax.annotation</groupId>
    <artifactId>jsr250-api</artifactId>
    <version>1.0</version>
</dependency>
```
**方法签名上使用**
```java
@RolesAllowed({"ROLE_ADMIN", "ROLE_USER"})
@DenyAll
@PermitAll
```
