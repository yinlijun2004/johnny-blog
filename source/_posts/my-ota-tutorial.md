---
title: 小白的全栈梦之从零搭建Android OTA系统（0）
date: 2017-12-20 17:45:28
tags: [express, react, nodejs]
---

### 起因
作为一个半路出家的码农，一个野生Android工程师，一直对<b>全栈</b>心怀憧憬，但未有行动。直到两年之前有机会开始接触javascript，写了一些前端的代码，加上nodejs社区这些年的蓬勃发展，我觉得这是一个机会。正如某位长者教导的:
>人的一生当然要靠自我奋斗,当然也要考虑历史的进程。

我觉得是时候了。

这是一个完整的项目，也许耗时会比较长，因为上班时间还有工作要完成，但是我会尽力完成它，我的目标是达到可以商用的水平，而不是一个玩具。实现的过程的文章会贴在[本站](http://www.yinlijun.com)，代码将会托管到github [android ota system](https://github.com/yinlijun2004/android_ota_system)。

### 目标
到这个项目完结的时候，能实现如下功能。

- 服务端
  - 版本管理后台
  - 用户管理后台
- PC端
  - 版本管理界面
  - 用户操作界面
- Android端
  - 查询版本
  - 下载版本
  - 升级版本 


### 我的技术背景

语言方面
- javascript 熟练度 30%

框架
- nodejs 熟练度 5%
- react 熟练度 10%

### 会使用到的库
- react
- express

### 目录
- {% post_link my-ota-tutorial-1 用户注册登录的后台实现 %}
- {% post_link my-ota-tutorial-2 用户注册登录前端界面实现 %}