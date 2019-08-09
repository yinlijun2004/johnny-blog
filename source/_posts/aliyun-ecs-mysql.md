---
title: 远程访问阿里云ECS MYSQL服务的方法
date: 2019-06-21 11:07:33
tags: [mysql, ecs]
---

在连接阿里云ECS的Mysql服务的时候，碰到失败的问题，排查的步骤如下：

- 在云服务器实例的安全组入规则里面，开通MYSQL(3306)端口，如果你的MYSQL服务绑定的是其他端口，那么请开通对应端口。

- 在MYSQL的配置文件里面，我的ECS的配置文件是/etc/mysql/my.cnf里，注释掉
```
bind-address		= 127.0.0.1
```
或者改为
```
bind-address		= 0.0.0.0
```

- 将host字段的值改为%就表示在任何客户端机器上能以root用户登录到mysql服务器，建议在开发时设为%。   

```
mysql> use mysql;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> grant all privileges  on *.* to root@'%' identified by "password";
```
- 退出，然后重启mysql服务:
```bash
service mysql restart
```

然后就可以用MySQLWorkbench之类的工具远程连接数据库了。

或者在spring boot项目application.yml里面，配置远程连接：
```
spring:
  datasource:
    driver-class-name: com.mysql.jdbc.Driver
    username: root
    password: root
    url: jdbc:mysql://xx.xx.xx.xx:3306/test
```