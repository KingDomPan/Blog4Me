title: Spring-Data-Jpa-Open-Session-In-View
date: 2014-12-07 20:06:39
categories: Spring
tags: [Spring, Data, Jpa, Hibernate, Seesion]
---

由于OOPS项目使用的数据库为mysql, 日前需要做自动装机的需求功能,
使用了开源项目razor[@github](https://github.com/puppetlabs/razor-server)来实现自动装机功能,
razor使用的数据库为postgresql9.X系列,  因此在web系统上使用的mysql就存在一个不平衡点了,
就是**razor端的数据如何获取**的问题.

<!-- more -->

---

针对以上的问题, 我们考虑了2个方案
- 使用 Quartz 定时从 razor 采集数据到 mysql
- 直接在web端建立远程数据源

---

我们的需求是在界面上定时刷新razor端监测的主机状态(包括Node, Tag, Policy, Repo),
通过调用razor的restful api来完成.
**完成这个想法是个很简单的任务**
**为此我对Spring-Quartz的支持做了一个封装, 并使用Apache Http Client 4.3.6 进行了单例的池化管理**
**我想说的是, 最后这个想法放弃了, 因为api返回的非关系数据在mysql上实现太难搞了**
**评估之后发现写起来的代码几乎都是在解析json上**

---

那么就直接建立远端数据源
**由于mysql本身已经使用hibernate jpa完成了一个数据源的绑定**


**对远端数据源的话**
- 重复一个代码
- 使用其他的方式进行CRDU操作

刚好不满足一直使用HibernateTemplate的操作, 最近看上了Spring-Data-Jpa的知识, 捣鼓了下, 总算搞定了
直接上Spring-Data-Jpa的配置上, 虽然不是本文的主题^_~, 后续补上
#### Spring-Data-Jpa的配置
**用到了 maven 和 properties文件**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xmlns:c="http://www.springframework.org/schema/c"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:jee="http://www.springframework.org/schema/jee"
    xmlns:lang="http://www.springframework.org/schema/lang"
    xmlns:task="http://www.springframework.org/schema/task"
    xmlns:util="http://www.springframework.org/schema/util"
    xmlns:mongo="http://www.springframework.org/schema/data/mongo"
    xmlns:cache="http://www.springframework.org/schema/cache"
    xmlns:jdbc="http://www.springframework.org/schema/jdbc"
    xmlns:jpa="http://www.springframework.org/schema/data/jpa"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:repository="http://www.springframework.org/schema/data/repository"
    xmlns:tx="http://www.springframework.org/schema/tx"
    xsi:schemaLocation="http://www.springframework.org/schema/jee
    http://www.springframework.org/schema/jee/spring-jee-3.0.xsd
        http://www.springframework.org/schema/lang
        http://www.springframework.org/schema/lang/spring-lang-3.0.xsd
        http://www.springframework.org/schema/data/mongo
        http://www.springframework.org/schema/data/mongo/spring-mongo-1.5.xsd
        http://www.springframework.org/schema/jdbc
        http://www.springframework.org/schema/jdbc/spring-jdbc-3.2.xsd
        http://www.springframework.org/schema/data/repository
        http://www.springframework.org/schema/data/repository/spring-repository-1.7.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc-3.2.xsd
        http://www.springframework.org/schema/task
        http://www.springframework.org/schema/task/spring-task-3.0.xsd
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
        http://www.springframework.org/schema/data/jpa
        http://www.springframework.org/schema/data/jpa/spring-jpa-1.3.xsd
        http://www.springframework.org/schema/cache
        http://www.springframework.org/schema/cache/spring-cache-3.2.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx-3.2.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop-3.0.xsd
        http://www.springframework.org/schema/util
        http://www.springframework.org/schema/util/spring-util-3.0.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context-3.0.xsd">

    <context:property-placeholder location="classpath:jpa-resources.properties"/>

    <!-- razor数据源 -->
    <bean id="razorDataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource"
        destroy-method="close">
        <property name="driverClass" value="${jpa.connection.driver}" />
        <property name="jdbcUrl" value="${jpa.connection.url}" />
        <property name="user" value="${jpa.connection.username}" />
        <property name="password" value="${jpa.connection.password}" />
        <property name="minPoolSize" value="10"/>
        <property name="maxPoolSize" value="120"/>
        <property name="maxIdleTime" value="1800"/>
        <property name="acquireIncrement" value="10"/>
        <property name="maxStatements" value="2"/>
        <property name="initialPoolSize" value="2"/>
        <property name="idleConnectionTestPeriod" value="1800"/>
        <property name="acquireRetryAttempts" value="1"/>
        <property name="breakAfterAcquireFailure" value="true"/>
        <property name="testConnectionOnCheckout" value="false"/>
    </bean>

    <!-- jpa Entity Factory 配置 -->
    <bean id="entityManagerFactory"
        class="org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean">
        <property name="dataSource" ref="razorDataSource" />
        <property name="packagesToScan">
            <list>
                <value>com.ffcs.itm.oops.web.autopacker.entity</value>
            </list>
        </property>
        <property name="persistenceUnitName" value="${jpa.persistenceUnitName}"/>
        <property name="persistenceProvider">
            <bean class="org.hibernate.ejb.HibernatePersistence"/>
        </property>
        <property name="jpaVendorAdapter">
            <bean class="org.springframework.orm.jpa.vendor.HibernateJpaVendorAdapter">
                <property name="generateDdl" value="${jpa.generateDdl}"/>
                <property name="database" value="${jpa.database}"/>
                <property name="databasePlatform" value="${jpa.databasePlatform}"/>
                <property name="showSql" value="${jpa.showSql}"/>
            </bean>
        </property>
        <property name="jpaDialect">
            <bean class="org.springframework.orm.jpa.vendor.HibernateJpaDialect"/>
        </property>
        <property name="jpaPropertyMap">
            <map>
                <!-- 使用自定义的validator进行jsr303验证 -->
                <entry key="javax.persistence.validation.factory" value-ref="validator"/>
                <!-- 只扫描class文件，不扫描hbm，默认两个都搜索 -->
                <entry key="hibernate.archive.autodetection" value="class"/>
                <!-- 不检查@NamedQuery -->
                <entry key="hibernate.query.startup_check" value="false"/>
                <entry key="hibernate.query.substitutions"
                    value="${hibernate.query.substitutions}"/>
                <entry key="hibernate.default_batch_fetch_size"
                    value="${hibernate.default_batch_fetch_size}"/>
                <entry key="hibernate.max_fetch_depth"
                    value="${hibernate.max_fetch_depth}"/>
                <entry key="hibernate.generate_statistics"
                    value="${hibernate.generate_statistics}"/>
                <entry key="hibernate.bytecode.use_reflection_optimizer"
                    value="${hibernate.bytecode.use_reflection_optimizer}"/>
                <entry key="hibernate.cache.use_second_level_cache"
                    value="${hibernate.cache.use_second_level_cache}"/>
                <entry key="hibernate.cache.use_query_cache"
                    value="${hibernate.cache.use_query_cache}"/>
                <entry key="hibernate.cache.region.factory_class"
                    value="${hibernate.cache.region.factory_class}"/>
                <!-- <entry key="net.sf.ehcache.configurationResourceName" value="${net.sf.ehcache.configurationResourceName}"/> -->
                <entry key="hibernate.cache.use_structured_entries"
                    value="${hibernate.cache.use_structured_entries}"/>
            </map>
        </property>
    </bean>

    <!--事务管理器配置-->
    <bean id="transactionManager"
        class="org.springframework.orm.jpa.JpaTransactionManager">
        <property name="entityManagerFactory" ref="entityManagerFactory"/>
    </bean>

    <tx:annotation-driven transaction-manager="transactionManager" />

    <tx:advice id="transactionAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <!--hibernate4必须配置为开启事务 否则 getCurrentSession()获取不到-->
            <tx:method name="get*" propagation="REQUIRED" read-only="true"/>
            <tx:method name="count*" propagation="REQUIRED" read-only="true"/>
            <tx:method name="find*" propagation="REQUIRED" read-only="true"/>
            <tx:method name="list*" propagation="REQUIRED" read-only="true"/>
            <tx:method name="*" propagation="REQUIRED"/>
        </tx:attributes>
    </tx:advice>

    <aop:config expose-proxy="true" proxy-target-class="true">
        <!-- 只对业务逻辑层实施事务 -->
        <aop:pointcut id="transactionPointcut"
            expression="execution(* com.ffcs.itm.oops..serviced..*+.*(..))"/>
        <aop:advisor id="transactionAdvisor"
            advice-ref="transactionAdvice" pointcut-ref="transactionPointcut"/>
    </aop:config>

    <!-- 扫描 spring data jpa repository -->
    <jpa:repositories
        base-package="com.ffcs.itm.oops.**.repository"
        repository-impl-postfix="Impl"
        entity-manager-factory-ref="entityManagerFactory"
        factory-class="com.ffcs.itm.oops.common.repository.support
                            .SimpleBaseRepositoryFactoryBean"
        transaction-manager-ref="transactionManager">
    </jpa:repositories>

    <!--设置RepositoryUtil辅助类所需的entityManagerFactory-->
    <bean class="org.springframework.beans.factory.config.MethodInvokingFactoryBean">
        <property name="staticMethod"
                  value="com.ffcs.itm.oops.common.repository.support
                    .RepositoryUtil.setEntityManagerFactory"/>
        <property name="arguments" ref="entityManagerFactory"/>
    </bean>

</beans>
```


配置成功之后, 2个数据源终于不冲突可以好好使用了
- 之前以为Spring-Data-Jpa只支持Hibernate4, 项目使用3, 换得一阵惨痛

#### 本文重点

搭建成功之后, 测试了一下数据, 结果先后来个以下2个问题
- OpenSessionInView的问题, No Session or Session closed;
- JSONObject的转换问题了, 嵌套太多, 死循环了(要么DTO, 要么注册过滤器, 说多了都是泪)

#### OpenSessionInView配置
**一下子慌了, 本来web.xml**是有配置下列代码的, 没想到没通用
```xml
<!-- Hibernate 过滤器 -->
<filter>
    <filter-name>openSessionInView</filter-name>
    <filter-class>
        org.springframework.orm.hibernate3.support.OpenSessionInViewFilter
    </filter-class>
    <init-param>
        <param-name>flushMode</param-name>
        <param-value>AUTO </param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>openSessionInView</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

所幸的是, Spring-Data-Jpa也有一个这样的配置, 还好没出大事情
```xml
<!-- Spring Data Jpa 防止Session失效设置  -->
<filter>
    <filter-name>openEntityManagerInViewFilter</filter-name>
    <filter-class>
        org.springframework.orm.jpa.support.OpenEntityManagerInViewFilter
    </filter-class>
    <init-param>
        <param-name>entityManagerFactoryBeanName</param-name>
        <param-value>entityManagerFactory</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>openEntityManagerInViewFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```