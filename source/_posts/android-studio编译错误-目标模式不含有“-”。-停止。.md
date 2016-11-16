---
title: 'android studio编译错误: *** 目标模式不含有“%”。 停止。'
date: 2016-11-5 18:04:19
tags: [android studio, android]
---

Android Studio NDK编译出现如下错误：
*** target pattern contains no `%'. Stop
中文：
*** 目标模式不含有“%”。 停止。
可能是obj目录的问题，需要删掉。
在工程目录下find所有的obj目录
```bash
find . -name obj
```