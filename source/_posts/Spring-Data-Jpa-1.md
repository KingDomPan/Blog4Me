title: Spring-Data-Jpa-基础概括
date: 2014-12-11 09:05:35
categories: Spring-Data
tags: [Spring, Data, Jpa]
---

前些天对Spring-Data-Jpa的基础知识做了点扫盲, 现在随手记录下, 主要是一些关键点.
具体的可以查看官网[Spring-Data-Jpa](http://docs.spring.io/spring-data/jpa/docs/1.7.1.RELEASE/reference/html/)

#### Spring-Data-Jpa接口
- Repository : 最顶层的空接口, 统一所有的 Repository 类型
- CrudRepository : Repository 的子接口, 提供 CRDU 的功能
- PagingAndSortingRepository : CrudRepository 的子接口, 添加分页和排序功能
- JpaRepository : PagingAndSortingRepository 的子接口, 增加一些实用的功能, 如批量操作
- JpaSpecificationExecutor : 用来做负责查询的接口
- Specification : 查询规范, 要做复杂的查询, 只需围绕这个规范来设置查询条件即可

<!-- more -->

#### Spring-Data-Jpa简单配置
- 声明DAO接口, 继承JpaRepository接口即可
- 编写Service, 注入DAO接口(Spring-Data-Jpa会通过代理创建接口对象)

#### Spring-Data-Jpa基本功能
- Pageable 接口
- Sort 类
- Order 类
- JpaRepository 的查询功能, 符合规范的直接定义在接口中
![](/images/Spring-Data/keys.jpg)

#### Spring-Data-Jpa方法命名解析
1. 去除多余的前缀, 如find findBy read readBy get getBy
2. 根据POJO规范, 首字母小写, 比如 findByUserDepUuid --> userDepUuid
3. 检测 userDepUuid 是否为属性, 是的话根据此属性进行查询
4. 否的话, 从右往左截取第一个大写字母开头的字符串此处为Uuid, 否的话重复第二步
5. 最后为 Doc.user.dep.uuid


- 特殊参数, 可在方法名上增加 排序或者分页的功能
```java
Page<UserModel> findByName(String name, Pageable pageable);
List<UserModel> findByName(String name, Sort sort);
```

- 也可以使用JPA的NamedQueries
    1. 在实体类上使用
           @NamedQuery(name="UserModel.findByAge", query="select o from UserModel o where o.age>?1")
    2. 自己实现的DAO的Repository接口里面定义一个同名方法.
           public List<UserModel> findByAge(int age);
    3. Spring会先找是否有同名的NamedQuery,如果有, 那么就不会按照接口定义的方法来解析

- 可以在自定义的查询方法上使用 @Query 来指定该方法要执行的查询语句.
        @Query("select o from UserModel o where o.uuid=?1")
        public List<UserModel> findByUuidOrAge(int uuid);
        1.方法的参数个数必须和@Query里面需要的参数个数一致
        2.如果是like, 后面的参数需要前面或者后面加“%” (%?1%)

- @Query 指定本地查询
        @Query(value="select * from tbl_user where name like %?1" , nativeQuery=true)
        public List<UserModel> findByUuidOrAge(String name);

- @Param 使用命名化参数
        @Query(value="select o from UserModel o where o. name like %:nn")
        public List<UserModel> findByUuidOrAge(@Param("nn") String name);

- 更新类的Query语句, 添加 @Modifying
        @Modifying
        @Query(value="update UserModel o set o.name=:newName where o.name like %:nn")
        public int findByUuidOrAge(@Param("nn") String name, @Param("newName") String newName);

#### 查询规范的指定查询策略
`<jpa:repositories>` `query-lookup-strategy`
- create-if-not-found: 如果方法通过@Query指定了查询语句, 则使用该语句实现          查询: 如果没有, 则查找是否定义了符合条件的命名查询,如果找到, 则使用该                 命名查询: 如果两者都没有找到, 则通过解析方法名字来创建查询.                         这是 querylookup-strategy 属性的默认值
- create: 通过解析方法名字来创建查询.即使有符合的命名查询, 或者方法通过               @Query指定的查询语句, 都将会被忽略
- use-declared-query: 同 create-if-not-found, 如果两者都没有找到, 则抛出异常

#### 扩展 JpaRepository
- 不需要实现接口, 只要编写接口同名类+Impl即可, spring会自动扫描
- 接口中编写方法
        public Page<Object[] > getByCondition(UserQueryModel u);
- 实现类实现方法, 会被自动找到

#### Specifications 查询
**基于 JPA Criteria 查询, 这是一种类型安全和更面向对象的查询**
- 接口方法
        Predicate toPredicate(Root<T> root, CriteriaQuery<?> query,
        CriteriaBuilder cb);
- Criteria 查询基本概念
    1. Criteria 查询是以元模型的概念为基础的, 元模型是为具体持久化单元的受管           实体定义的, 这些实体可以是实体类, 嵌入类或者映射的父类
    2. CriteriaQuery接口: 代表一个specific的顶层查询对象, 它包含着查询的各个       部分, 比如:  select ; from; where; group by; order by等
    3. Root接口: 代表Criteria查询的根对象, Criteria查询的查询根定义了实体类         型,能为将来导航获得想要的结果, 它与SQL查询中的FROM子句类似