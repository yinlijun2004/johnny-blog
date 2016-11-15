---
title: ubuntu利用update-alternatives配置java
date: 2016-11-15 19:32:21
tags: ubuntu; update-alternatives
---

利用Android Studio开发，经常会碰到JDK版本的切换问题，安装好新版本的[jdk](http://www.oracle.com/technetwork/java/javase/downloads/index.html)之后，需要先配置到可选项。
```bash
yinlijun@sj:~$ sudo update-alternatives --install /usr/bin/java java /opt/jdk1.8.0_101/bin/java 100
update-alternatives: 警告: /etc/alternatives/java has been changed (manually or by a script); switching to manual updates only
yinlijun@sj:~$ sudo update-alternatives --install /usr/bin/javac javac /opt/jdk1.8.0_101/bin/javac 100
```

然后，选择默认的JDK版本：
```bash
yinlijun@sj:~$ sudo update-alternatives --config java
有 3 个候选项可用于替换 java (提供 /usr/bin/java)。

  选择       路径                                          优先级  状态
------------------------------------------------------------
  0            /opt/jdk1.6.0_37/bin/java                        10000     自动模式
  1            /opt/jdk1.6.0_37/bin/java                        10000     手动模式
  2            /opt/jdk1.8.0_101/bin/java                       100       手动模式
  3            /usr/lib/jvm/java-7-openjdk-amd64/jre/bin/java   1051      手动模式

要维持当前值[*]请按回车键，或者键入选择的编号：2
update-alternatives: using /opt/jdk1.8.0_101/bin/java to provide /usr/bin/java (java) in 手动模式
yinlijun@sj:~$ sudo update-alternatives --config javac
有 3 个候选项可用于替换 javac (提供 /usr/bin/javac)。

  选择       路径                                       优先级  状态
------------------------------------------------------------
  0            /opt/jdk1.6.0_37/bin/javac                    10000     自动模式
  1            /opt/jdk1.6.0_37/bin/javac                    10000     手动模式
  2            /opt/jdk1.8.0_101/bin/javac                   100       手动模式
* 3            /usr/lib/jvm/java-7-openjdk-amd64/bin/javac   1051      手动模式

要维持当前值[*]请按回车键，或者键入选择的编号：2
```
