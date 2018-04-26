---
title: ubuntu 16.04 编译android填坑记录
date: 2018-04-26 17:05:30
tags: [ubuntu, android, jdk, make]
---

实在受不了ubuntu 12.04了，于是更新到了16.04，但是重新编译android ASOP的时候出了一些状况，在此记录一下。

#### java丢失

重新安装openJDK 8

```bash
sudo apt-get install openjdk-8-jre
sudo apt-get install openjdk-8-jdk-headless
```
<!-- more -->

不要安装openJDK 9，会报错。

#### make版本变成了4.1

编译的时候报错:
```
build/core/prebuilt.mk:91: *** recipe commences before first target. Stop.
```

这是因为make版本错误，下载make v3.81代码编译。打开 [http://ftp.gnu.org/gnu/make/](http://ftp.gnu.org/gnu/make/)

选择make-3.81.tar.gz下载，执行解压、编译、安装步骤：
```
tar zxvf make-3.81.tar.gz
cd make-3.81
sudo ./configure 
sudo make 
sudo make install
```

执行完以后，重新开一个终端，检查make版本。
```
make -v
```
成功就会提示
```
GNU Make 3.81
Copyright (C) 2006  Free Software Foundation, Inc.
This is free software; see the source for copying conditions.
There is NO warranty; not even for MERCHANTABILITY or FITNESS FOR A
PARTICULAR PURPOSE.
```

