---
title: mac-dubbo-admin
date: 2020-03-20 11:39:48
tags:
---

## zookeeper
### 下载
网页：https://zookeeper.apache.org/releases.html

将release版本下载到/user/local下

### 修改默认配置及默认端口
```bash
cd /usr/local/apache-zookeeper-3.6.0-bin
mv zoo-sample.cfg zoo.cfg
```
添加如下行
```bash
admin.serverPort=8888
```

## dubbo

进入 dubbo-admin-ui 目录
```bash
cd dubbo-admin-ui

sudo npm install
```
回到上一级目录，执行mvn打包
```bash
cd ..
sudo mvn clean package;
```

```bash
[INFO] 
[INFO] dubbo-admin ........................................ SUCCESS [  2.231 s]
[INFO] dubbo-admin-ui ..................................... FAILURE [  1.624 s]
[INFO] dubbo-admin-server ................................. SKIPPED
[INFO] dubbo-admin-distribution ........................... SKIPPED
[INFO] ------------------------------------------------------------------------
[INFO] BUILD FAILURE
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  4.179 s
[INFO] Finished at: 2020-03-20T12:33:10+08:00
[INFO] ------------------------------------------------------------------------
[ERROR] Failed to execute goal com.github.eirslett:frontend-maven-plugin:1.6:install-node-and-npm (install node and npm) on project dubbo-admin-ui: Could not extract the Node archive: Could not extract archive: '/var/root/.m2/repository/com/github/eirslett/node/9.11.1/node-9.11.1-darwin-x64.tar.gz': EOFException -> [Help 1]
[ERROR] 
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR] 
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/MojoFailureException
[ERROR] 
[ERROR] After correcting the problems, you can resume the build with the command
[ERROR]   mvn <goals> -rf :dubbo-admin-ui

```

删除node安装包
```bash
sudo rm /var/root/.m2/repository/com/github/eirslett/node/9.11.1/node-9.11.1-darwin-x64.tar.gz
```

重新安装
```bash
sudo mvn clean package;
```

安装成功的输出
```bash
[INFO] Executing tasks

main:
     [copy] Copying 1 file to /usr/local/incubator-dubbo-admin/dubbo-admin-distribution/target
[INFO] Executed tasks
[INFO] ------------------------------------------------------------------------
[INFO] Reactor Summary for dubbo-admin 0.1:
[INFO] 
[INFO] dubbo-admin ........................................ SUCCESS [  1.634 s]
[INFO] dubbo-admin-ui ..................................... SUCCESS [02:22 min]
[INFO] dubbo-admin-server ................................. SUCCESS [07:21 min]
[INFO] dubbo-admin-distribution ........................... SUCCESS [ 12.315 s]
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
[INFO] Total time:  09:58 min
[INFO] Finished at: 2020-03-20T12:46:00+08:00
[INFO] ------------------------------------------------------------------------
```

修改dubbo服务后台端口
```
sudo vim src/main/resources/application.properties
```
加入
```
server.port=8087
```

可以修改前端默认端口(8087)
```bash
vim config/index.js 
```

运行dubbo
```bash
sudo mvn --projects dubbo-admin-server spring-boot:run
```

dubbo admin网页新增配置
```
dubbo.registry.address=zookeeper://127.0.0.1:2181 
dubbo.metadata-report.address=zookeeper://127.0.0.1:2181
```

在提供端配置
```
dubbo:
  application:
    name: wallet-user-provider
  protocol:
    name: dubbo
    port: 20880
  registry:
    address: zookeeper://localhost:2181
  provider:
    timeout: 1000
  metadata-report:
    address: zookeeper://127.0.0.1:2181
```

参考
> https://github.com/apache/dubbo-admin
> https://blog.csdn.net/m_vaw/article/details/89475565