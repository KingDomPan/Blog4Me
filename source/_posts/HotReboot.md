title: JRebel-告诉你什么是真正的热部署
date: 2014-12-14 15:23:36
categories: JRebel
tags: [Maven, JRebel]
---

以前每次修改java代码时, 或者修改了Spring, Struts2之类的配置文件时,  都需要重启JAVA Web容器,
才能使修改后的代码或者配置生效,  Tomcat或者Jetty都是如此,  在使用Weblogic的时候就算修改了weblogic.xml也是没效果,  查找了很多资料,  都是千篇一律的.


前几天调试OOPS-即时通信模块的api接口时, 要经常修改java代码来调试, 结果就是简简单单的2个接口,  由于不停的修改, 启动, 搞了一个下午, 快崩溃了.  这实在不是我们该干的事啊.

后来谷歌了一下, 不小心就搜索到了JRebel这个东西, 实在是码农的福音啊. 不说了, 两步教你怎么配置.

<!-- more -->

##### Maven的jetty插件
```xml
<!-- 使用jetty进行调试, mvn jetty:run -->
<plugin>
    <groupId>org.eclipse.jetty</groupId>
    <artifactId>jetty-maven-plugin</artifactId>
    <version>${jetty.version}</version>
    <configuration>
        <httpConnector>
            <port>9090</port>
        </httpConnector>
        <scanIntervalSeconds>1</scanIntervalSeconds>
        <reload>automatic</reload>
        <war>${project.build.directory}/${project.build.finalName}</war>
        <webApp>
            <contextPath>/${project.build.finalName}</contextPath>
        </webApp>
    </configuration>
    <executions>
        <execution>
            <id>start-jetty</id>
            <phase>pre-integration-test</phase>
            <goals>
                <goal>run</goal>
            </goals>
            <configuration>
                <scanIntervalSeconds>0</scanIntervalSeconds>
                <daemon>true</daemon>
            </configuration>
        </execution>
        <execution>
            <id>stop-jetty</id>
            <phase>post-integration-test</phase>
            <goals>
                <goal>stop</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

##### 下载JRebel.jar(以下方式2选1)
**注意: JRebel不是开源的, 要收费的**
- 直接下载中国版的JRebel, 你懂的
- 直接在Eclipse等IDE安装JRebel插件

##### Eclipse的Maven配置
![](/images/jrebel/1.jpg)
**主要是配置虚拟机的启动参数**
```bash
-noverify -javaagent:E:\bin\jrebel.jar
-Drebel.spring_plugin=true
-Xms256M -Xmx512M -XX:MaxPermSize=128m
```

##### 启动看看
**看到熟悉的配置了吧**
![](/images/jrebel/2.jpg)