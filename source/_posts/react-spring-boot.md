---
title: 如何将react项目部署到spring boot项目
date: 2019-06-19 17:50:19
tags: react, spring boot
---

现在主流的开发方式是前后端分离，前端只管界面开发，后端只提供rest api接口，那如何将react构建好的网页部署到spring boot项目呢。

首先，在spring boot项目的pom.xml里面要加入构建的插件。
```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
</plugin>
<plugin>
    <groupId>com.github.eirslett</groupId>
    <artifactId>frontend-maven-plugin</artifactId>
    <version>1.6</version>
    <configuration>
        <workingDirectory>[前端项目路径]</workingDirectory>
        <installDirectory>target</installDirectory>
    </configuration>
    <executions>
        <execution>
            <id>install node and npm</id>
            <goals>
                <goal>install-node-and-npm</goal>
            </goals>
            <configuration>
                <nodeVersion>v8.11.3</nodeVersion>
                <npmVersion>5.6.0</npmVersion>
            </configuration>
        </execution>
        <execution>
            <id>npm install</id>
            <goals>
                <goal>npm</goal>
            </goals>
            <configuration>
                <arguments>install</arguments>
            </configuration>
        </execution>
        <execution>
            <id>build front end</id>
            <goals>
                <goal>npm</goal>
            </goals>
            <configuration>
                <arguments>[前端构建脚本]</arguments>
            </configuration>
        </execution>
    </executions>
</plugin>
<plugin>
    <artifactId>maven-antrun-plugin</artifactId>
    <executions>
        <execution>
            <phase>generate-resources</phase>
            <configuration>
                <target>
                    <copy todir="${project.build.directory}/classes/static">
                        <fileset dir="[前端构建结果路径]" />
                    </copy>
                </target>
            </configuration>
            <goals>
                <goal>run</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

上面有三个地方要注意：
```
<workingDirectory>[前端项目路径]</workingDirectory>
```
这里<b>前端项目路径</b>填你自己的前端路径。

```
<arguments>[前端构建脚本]</arguments>
```
<b>前端构建脚本</b>，填你前段项目的package.json下的构建脚本就行了，比如说'run build'

```
<fileset dir="[前端构建结果路径]" />
```
这里的<b>前端构建结果路径</b>，填前端构建结果的路径，一般在<b>前端项目路径</b>下。


然后，在spring boot目录下，执行命令：
```bash
mvn clean install
```
这条命令，会将前端项目构建好，并且将构建结果拷贝到spring boot的target/classes/static下面。

然后运行spring boot项目，就能看到react项目可以渲染出来了。

如果你的react是多页应用，并且又用了shiro等权限框架，可能要配置一下匿名访问的路径了，因为请求网页的路径，会要求登录。需要在application.yml里面修改shiro配置：
```
  shiro:
    anonUrl: /**/*.html,/css/**,/js/**,/fonts/**,/img/**
```
