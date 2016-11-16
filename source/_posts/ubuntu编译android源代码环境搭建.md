---
title: ubuntu编译android 6.0源代码环境搭建
tags:
  - ubuntu
  - android
  - 环境搭建
date: 2016-11-16 09:04:37
---


## 安装jdk
```bash
sudo apt-get install openjdk-7-jdk openjdk-7-jre 
```
如果之前系统是其他版本的JDK，需要把环境变量切换过来。

<!--more-->

jdk版本错误会导致编译错误，如
```
prebuilts/sdk/api/23.txt:41822: error 9: Removed public constructor android.widget.Toolbar.LayoutParams.Toolbar.LayoutParams(LayoutParams)
prebuilts/sdk/api/23.txt:41823: error 9: Removed public constructor android.widget.Toolbar.LayoutParams.Toolbar.LayoutParams(MarginLayoutParams)
prebuilts/sdk/api/23.txt:41824: error 9: Removed public constructor android.widget.Toolbar.LayoutParams.Toolbar.LayoutParams(LayoutParams)
prebuilts/sdk/api/23.txt:42895: error 9: Removed public constructor java.io.ObjectInputStream.GetField.ObjectInputStream.GetField()
prebuilts/sdk/api/23.txt:42955: error 9: Removed public constructor java.io.ObjectOutputStream.PutField.ObjectOutputStream.PutField()
prebuilts/sdk/api/23.txt:43623: error 9: Removed public constructor java.lang.Character.Subset.Character.Subset(String)
prebuilts/sdk/api/23.txt:46730: error 9: Removed public constructor java.nio.channels.Pipe.SinkChannel.Pipe.SinkChannel(SelectorProvider)
prebuilts/sdk/api/23.txt:46735: error 9: Removed public constructor java.nio.channels.Pipe.SourceChannel.Pipe.SourceChannel(SelectorProvider)
prebuilts/sdk/api/23.txt:47370: error 9: Removed public constructor java.security.KeyStore.Builder.KeyStore.Builder()
prebuilts/sdk/api/23.txt:47379: error 9: Removed public constructor java.security.KeyStore.CallbackHandlerProtection.KeyStore.CallbackHandlerProtection(CallbackHandler)
prebuilts/sdk/api/23.txt:47391: error 9: Removed public constructor java.security.KeyStore.PasswordProtection.KeyStore.PasswordProtection(char)
```

## 安装其他工具包
```bash
sudo apt-get install git gitg gnupg flex bison gperf build-essential  zip curl libc6-dev  libncurses5-dev:i386 x11proto-core-dev  libx11-dev:i386 libreadline6-dev:i386   libgl1-mesa-dev g++-multilib mingw32 tofrodos python-markdown	libxml2-utils xsltproc zlib1g-dev:i386 libarchive-zip-perl 
```

