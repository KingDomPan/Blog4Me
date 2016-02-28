title: 基于Fiddler的前端调试技术
date: 2014-12-09 19:04:06
categories: HTTP
tags: [Fiddler, JS, HTTP]
---

本文是我在浙江现场出差的时候, E周E文给现场的兄弟们讲解开发技术
草拟的讲义, 文后奉上了录制视频, 主要讲解JAVA WEB开发的知识, 并
重点通过Fiddler这个前台调试神器现场演示了作用.

**视频地址** [怒点我](http://pan.baidu.com/s/1gdkXFtp)

<!-- more -->

### Best Practice 2 ME 4 WEB DEV

#### JAVA WEB
** 主要讲述了javaweb的开发知识, 并且涉猎了其他的web开发 **
- WEB:ASP. NET, JSP, PHP, RUBY, PYTHON
- J2EE: [J2EE](http://www.importnew.com/10716.html)
- WEB前端涉及技术
    + HTML (规范网站的结构和内容)
    + CSS(规范风格)
    + JS(操作DOM)(框架:extjs,jquery等等),Node.js服务端的js框架,高性能NODEJS
- JAVA:struts1(2), hibernate,mybatis,spring


#### 前端工程师
**忘了当时讲什么了, 见连接[点我点我](http://www.admin10000.com/document/4216.html#rd)**

- 程序员修炼三部曲
    + 版本控制:CSV SVN(集中) GIT(分布式)
    + 单元测试:DTD(Junit, TestNG)
    + 项目构建:ANT MAVEN(jar)


#### 搭建一个web项目
** 主要是讲解路由映射的概念, 方便后面实例讲解**
- 静态资源:没有urlrewrite的话,直接相对项目路径来查找
- Servlet:路径映射在web.xml找(注解配置和Restful风格的配置令论)


#### 前台静态文件调试(HTML,CSS),脚本调试(JS)

- HTTP调试
- JS调试
- 调试利器
    + IE开发利器-IE10中的F12开发者工具[IE10-F12使用详解](http://wlzcool.blog.51cto.com/5341447/1201305)
    + FIDDLER2[基本介绍](http://www.telerik.com/fiddler) **注意.net版本问题**
    + FIDDLER2基本介绍2[基本介绍2](http://www.cnblogs.com/TankXiao/archive/2012/02/06/2337728.html)


#### Fiddler2实战
- 代码定位(MyEclipse[高级]查找)
    + 自定义查询表格(数据请求及其格式)
    + 未处理事务(/workshop/queryTemplate/main.html?id=511)
    + 查找传阅人员(/workshop/form/index.jsp?fullscreen=yes&flowId=570002005850&flowMod=11143&system_code=G)
    + 定位Ext.Window对象(/workshop/jdbcGather/jdbcGather.html)
    + 待办数不明确![](/images/Fiddler/1.png)
    + 需求流程的需求类型数据(/workshop/form/index.jsp?flowMod=11143)
- 拦截请求
    + 修改请求数据,删除数据库数据(需求黑名单)(/workshop/form/zjFormFile/zj_require_black.html)
- 拦截响应
    + 高额签到表,用户状态信息展(/workshop/form/zjFormFile/zj_sign.jsp?imsi=460030135885703&rowid=AAFu/qAHxAAABHzAAA)
- AutoResponder
    + 百度首页图片
    + IE:tree的多选解决思路
- 构建请求
    + 模拟登录ITNM系统,删除流程附件

#### 其他
**主要介绍了sublime和python**
- Sublime Text
    + Sublime Text [Sublime Text](http://www.sublimetext.com/)
    + PACKAGE CONTROL [package control](https://sublime.wbond.net/installation#)
- Python[运维, 游戏, web, 数据分析]
     + 上传集群服务器
     + 补丁文件比对工具