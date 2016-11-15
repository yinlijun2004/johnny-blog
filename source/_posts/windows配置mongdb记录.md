---
title: windows配置mongdb记录
date: 2016-11-9 17:55:51
tags: windows, mongodb
---

刚给windows电脑配置了mongdb记录一下。

- 前往[官网](https://www.mongodb.com/download-center)下载合适的安装包，我选择的是msi安装包，也可以选择下载zip包。

- 运行msi安装包（默认安装在C盘，没找到在哪里可以修改盘符）。

- 在D:\下创建好相关文件夹
![这里写图片描述](http://img.blog.csdn.net/20161109194149759)

<!--more-->

- 打开命令提示符，进入到mongodb安装文件夹下的bin目录
![这里写图片描述](http://img.blog.csdn.net/20161109194331090)

- 输入如下命令，注册MongoDB服务 

```bash
 mongod.exe --logpath D:\MongoDB\data\log\MongoDB.log --logappend --dbpath D:\MongoDB\data\db --directoryperdb --storageEngin=mmappv1 --serviceName MongoDB --install
```

![这里写图片描述](http://img.blog.csdn.net/20161109194436279)

- 启动MongoDB服务

```bash
 net start MongoDB
```

![这里写图片描述](http://img.blog.csdn.net/20161109194520874)
