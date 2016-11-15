---
title: 如何下载安装openJDK
date: 2016-11-15 20:04:44
tags: openjdk; java
---

## JDK 8

### **Debian, Ubuntu**
```
$ sudo apt-get install openjdk-8-jre
```
openjdk-8-jre只包含运行时环境(Java Runtime Environment）。如果你想开发java程序，请安装openjdk-8-jdk。

<!--more-->

### **Fedora, Oracle Linux, Red Hat Enterprise Linux**
```
$ su -c "yum install java-1.8.0-openjdk"
```
java-1.8.0-openjdk只包含运行时环境(Java Runtime Environment）。如果你想开发java程序，请安装java-1.8.0-openjdk-devel。

## JDK 7

### **Debian, Ubuntu**
```
$ sudo apt-get install openjdk-7-jre
```
openjdk-7-jre只包含运行时环境(Java Runtime Environment）。如果你想开发java程序，请安装openjdk-7-jdk。

### **Fedora, Oracle Linux, Red Hat Enterprise Linux**
```
$ su -c "yum install java-1.7.0-openjdk"
```
java-1.7.0-openjdk只包含运行时环境(Java Runtime Environment）。如果你想开发java程序，请安装java-1.7.0-openjdk-devel。

## JDK 6

### **Debian, Ubuntu**
```
$ sudo apt-get install openjdk-6-jre
```
openjdk-6-jre只包含运行时环境(Java Runtime Environment）。如果你想开发java程序，请安装openjdk-6-jdk。

### **Fedora, Oracle Linux, Red Hat Enterprise Linux**
```
$ su -c "yum install java-1.6.0-openjdk"
```
java-1.6.0-openjdk只包含运行时环境(Java Runtime Environment）。如果你想开发java程序，请安装java-1.6.0-openjdk-devel。


参考[http://openjdk.java.net/install/](http://openjdk.java.net/install/)