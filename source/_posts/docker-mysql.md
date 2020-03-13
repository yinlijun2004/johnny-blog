---
title: MAC下利用Docker安装Mysql
date: 2020-03-10 16:31:00
tags: [docker, mac, mysql]
---

昨天晚上电脑莫名其妙死机了，不得已按电源键重启，不料重启之后Mysql就一直启动不了。
搞了老半天没搞好，谷歌也古了，百度也百了，好像还没人碰到跟我一样的错误，一气之下把我MAC上的mysql删个干净。

### 删除mysql
```bash
sudo rm /usr/local/mysql
sudo rm -rf /usr/local/mysql*
sudo rm -rf /Library/StartupItems/MySQLCOM
sudo rm -rf /Library/PreferencePanes/My*
rm -rf ~/Library/PreferencePanes/My*
sudo rm -rf /Library/Receipts/mysql*
sudo rm -rf /Library/Receipts/MySQL*
sudo rm -rf /var/db/receipts/com.mysql.*
```
> 如果你不是默认路径安装的或者忘记了是不是默认路径安装的，那么除了执行上面的命令之外，还要检查以下文件中是否有对应的文件，有的话删除即可。

检查/usr/local/Cellar目录是否有mysql文件，有的话删除。
检查/usr/local/var 里的mysql文件，有的话删除。
检查/tmp 里的mysql.sock、mysql.sock.lock、 my.cnf文件，有的话删除。
err文件以及pid文件都是在/usr/local/var/mysql中，有的话删除。
brew安装的安装包存储在/usr/local/Library/Cache/Homebrew，有的话删除。
一定要记得执行brew cleanup。

删完我转念一想，现在docker挺火的，我不如直接在docker上安装mysql，说干就干。

### 安装docker
docker的安装暂不赘述。

[Docker下载链接](https://download.docker.com/mac/stable/Docker.dmg)

### docker下安装mysql

在docker hub上搜索[mysql](https://hub.docker.com/_/mysql)，可以自己选择版本，不选版本的话就是latest，即最新版本。

```
docker pull mysql:5.7
```

### 编排符合项目的mysql镜像

原生的mysql镜像不符合我的要求，因为我的项目有一些建表的操作，所以需要自己编排。

1. 在项目下创建docker目录。

2. 准备好两个建表的sql文件，分别为schema.sql，initial.sql，一般项目里面已经有了。

3. 创建prepare_db.sh脚本
```bash
#这个脚本是登录mysql并执行指定的sql文件

mysql -uroot -p$MYSQL_ROOT_PASSWORD <<EOF

#对应schema.sql
source $WORK_PATH/$SCHEMA_FILE; 

#对应initial.sql
source $WORK_PATH/$INITIAL_FILE;
``` 

4. 创建Dockerfile
```bash
#基础镜像使用的是mysql:5.7
FROM mysql:5.7

#定义工作目录变量
ENV WORK_PATH /usr/local/work

#定义会被容器自动执行的目录
ENV AUTO_RUN_DIR /docker-entrypoint-initdb.d

#定义要执行的sql文件名
ENV SCHEMA_FILE schema.sql
ENV INITIAL_FILE initial.sql

#定义要执行的shell文件名
ENV CREATE_DATA_SHELL prepare_db.sh

#创建工作文件目录
RUN mkdir -p $WORK_PATH

#把sql文件复制到工作目录下
COPY ./$FILE_0 $WORK_PATH/
COPY ./$FILE_1 $WORK_PATH/

#把要执行的shell文件放到/docker-entrypoint-initdb.d/目录下，容器会自动执行这个shell
COPY ./$CREATE_DATA_SHELL $AUTO_RUN_DIR/

#给执行文件增加可执行权限
RUN chmod a+x $AUTO_RUN_DIR/$CREATE_DATA_SHELL
```
值得一提的这个Dockerfile里的路径都是容器内的目录，并不是宿主机的目录，刚开始我傻夫夫的在宿主机里建了一堆目录。

5. 将上面的文件都移到docker文件夹下。

6. 生成镜像
```bash
sudo docker build -t t-xxx-mysql .
```
t-xxx-mysql表示定制化镜像名称

7. 生成容器
```bash
docker run --name xxx_mysql -v /path-to-mysql/mysql:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -p 33306:3306 -d t-xxx-mysql --character-set-server=utf8mb4 --collation-server=utf8mb4_unicode_ci
```

- xxx_mysql 表示镜像名称
- /path-to-mysql/mysql 表示宿主机的mysql数据目录映射到容器，可以防止容器删掉之后文件也丢失
- 33306 将该端口映射到3306端口，外部程序访问33306端口即可

如果是第一次运行，要保证/path-to-mysql/mysql是空目录，尤其之前有高版本的mysql的时候，如果里面不是空目录，会报错。

上面的prepare_db.sh脚本在容器第一次的时候会执行，但如果不是空目录，就不会执行。

8. 进入容器

```bash
docker exec -it xxx_mysql /bin/bash
```
如果prepare_db.sh没执行，可以在容器内手动执行脚本。
```bash
./docker-entrypoint-initdb.d/prepare_db.sh
```

9. 如果需要在容器内使用vim，采用如下方式安装
```bash
apt-get update
apt-get install vim
```

第一次使用docker，记录一下，错误之处，恳请大佬指正。

参考资料：
> https://www.jianshu.com/p/276c1271ae14 
> https://www.jianshu.com/p/1e4e465acb84